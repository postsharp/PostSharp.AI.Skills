---
uid: threading-performance
redirect_from:
  - threading/threading-performance
title: "Run-Time Performance of Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp threading models have minimal run-time overhead but high memory cost. They can significantly impact performance when instantiating large object graphs. To mitigate this, manually assign newly created objects to a concurrency controller."
---
# Run-Time Performance of Threading Model

When run-time verification of threading models is disabled (see <xref:threading-runtime-verification>), there is almost no run-time overhead of using PostSharp threading models compared to implementing thread synchronization manually, at least after the object model has been instantiated. 

However, PostSharp threading models come at a high memory cost, and there may be a significant performance overhead when instantiating large object graphs unless care is taken.


## Memory Consumption

Like most complex aspects implemented with PostSharp, threading models have a high memory cost. Several object instances are needed for each instance of a thread-safe class. If memory consumption is a concern, you should not use threading models on classes that have a very high number of instances.


## Instantiation of Large Object Trees

Threading models can have a significant impact on the cost of creating large object trees. In some situations, the cost of instantiating the tree can become `O(n^2)` instead of `O(n)`. The performance issue affects only the following threading models: 

* Synchronized,

* Reader-Writer Synchronized and

* Thread-Unsafe.

The performance issue stems from the fact that each root node in a tree needs its own instance of its concurrency controller.

Consider the scenario when you build a tree using a depth-first approach. That means that you would first instantiate the leaves of the tree, then the parent of the leaves, then the parent of the parent, and so on until you reach the root. Depth-first tree instantiation is a common strategy when you instantiate immutable trees. Note, however, that the immutable model is not affected by this issue.

When you start instantiating the leaves, and until the leaf is assigned to a parent, every leaf is the root of its own tree. This means that an instance of the concurrency controller may be created if needed. When you instantiate the first-level parents, a new concurrency controller is created for each first-level parent. When the leaf is assigned to its parent, the concurrency controllers of the leaves will be replaced by the concurrency controller of the immediate parent.

The same phenomenon occurs at each level of the tree. Whenever you assign a subtree to a parent, the concurrency controller of the whole subtree is reassigned. Replacing the concurrency controller of a subtree is an `O(n)` operation, and it must be performed for each of the `n` nodes, which means that in total the concurrency controllers will be reassigned `O(n^2)` times. 

During the operation of instantiating the tree, `O(n)` controllers may be instantiated. However, at the end of the operation, a single controller will remain in memory. 

To prevent PostSharp from allocating `O(n)` controllers and performing `O(n^2)` reassignments, you need to manually assign newly created objects to a concurrency controller. 


## Assigning the Concurrency Controller Manually

To avoid excessive creation and assignment of concurrency controllers, you can use the <xref:PostSharp.Patterns.Threading.ThreadingServices.WithConcurrencyController(PostSharp.Patterns.Threading.IConcurrencyController)> method to set the default controller for newly created objects. 

The following code snippet illustrates the use of <xref:PostSharp.Patterns.Threading.ThreadingServices.WithConcurrencyController(PostSharp.Patterns.Threading.IConcurrencyController)>. With this method, a single concurrency controller instance is created, and each node is assigned only once to this concurrency controller, amounting to 3 assignments for 3 nodes. Without the use of this method, 3 instances of the concurrency controller would have been created, and in total 5 assignments would be done. 

```csharp            
            using ( ThreadingServices.WithConcurrencyController( ConcurrencyControllerFactory.CreateSynchronizedController()) )
            {
                var child1 = new SynchronizedObject();
                var child2 = new SynchronizedObject();

                var parent = new SynchronizedObject();
                parent.Children.Add( child1 );
                parent.Children.Add( child2 );
            }
```


