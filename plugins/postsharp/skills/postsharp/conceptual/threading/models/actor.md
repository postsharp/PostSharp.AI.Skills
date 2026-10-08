---
uid: actor
redirect_from:
  - threading/actor
title: "Actor Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains the Actor Threading Model in PostSharp, which provides a race-free programming model by routing method calls to a single message queue processed by a single thread. It offers a guide on how to apply the model, examples, and potential constraints."
---
# Actor Threading Model

Given the complexity of trying to coordinate accesses to an object from several threads, sometimes it makes more sense to avoid multithreading altogether. The Actor model avoids the need for thread safety on class instances by routing method calls from each instance to a single message queue which is processed, in order, by a single thread.

Since the processing for each instance takes place in a single thread, multithreading is avoided altogether and the object is guaranteed to be free of data races. Calls are processed asynchronously in the order in which they were added to the message queue. Because all calls to an actor are asynchronous, it is recommended to use the async/await feature of C# 5.0.

In addition to providing a race-free programming model, the Actor pattern has the benefit of transparently distributing the computing load to all available CPUs without additional logic. Note that PostSharp’s implementation does not assign a new thread to each actor instance but uses a thread pool instead, so it is possible to have a very large number of actors with relatively low overhead.


## Applying the Actor pattern


To apply the Actor threading model:

1. Add the `PostSharp.Patterns.Threading` package to your project. 


2. Import the `PostSharp.Patterns.Threading` namespace into your file. 


3. Add the <xref:PostSharp.Patterns.Threading.ActorAttribute> to the class. 


4. Annotate your object model for parent/child relationships as described in <xref:aggregatable-adding>. 


5. It is recommended, but not required, that you change all methods to async methods, and modify the code that calls them.



### Example

Consider the following example of an `AverageCalculator` class. The code is not thread-safe because the `AddSample` method performs four operations (two reads and two writes) that must all be performed atomically. 

```csharp
class AverageCalculator
{
    float sum;
    int count;

    public void AddSample(float n)
    {
        this.count++;
        this.sum += n;
    }
    
    public float GetAverage()
    {
        return this.sum / this.count;
    }
}
```

We could use the Synchronized or Reader-Writer Synchronized threading model to make sure that the calling thread will wait if the object is currently being accessed by another thread. Another solution in this situation is to avoid concurrency altogether using the Actor pattern and asynchronous methods.

In the reworked example below, the `AverageCalculator` class has the <xref:PostSharp.Patterns.Threading.ActorAttribute> added, and the `GetAverage` method has been changed to an asynchronous method with the <xref:PostSharp.Patterns.Threading.ReentrantAttribute> attribute. The `AddSample` method was also changed to an async method returning `Task`, and the <xref:PostSharp.Patterns.Threading.ReentrantAttribute> attribute was applied. 

Note that we could keep the methods non-async, but it is good practice to make the public API of all actors async methods.

```csharp
[Actor]
    class AverageCalculator
    {
        float sum;
        int count;

        [Reentrant]
        public async Task AddSample(float n)
        {
            this.count++;
            this.sum += n;
        }

        [Reentrant]
        public async Task<float> GetAverage()
        {
            return this.sum / this.count;
        }
    }
```

You can now use the same `AverageCalculator` from two concurrent threads. 

```csharp
class Program
{
    static void Main(string[] args)
    {
        MainAsync().GetAwaiter().GetResult();
    }

    static async Task MainAsync()
    {
        AverageCalculator averageCalculator = new AverageCalculator();

        SampleObserver observer = new SampleObserver(averageCalculator);
        DataSources.Source1.Subscribe(observer);
        DataSources.Source2.Subscribe(observer);

        Console.ReadKey();

        float average = await averageCalculator.GetAverage();

        Console.WriteLine("Average: {0}", average);
    }
}
    
class SampleObserver : IObserver<float>
{
    AverageCalculator calculator;

       
    public void OnNext( float value )
    {
      // Each of the data sources can call us from a different thread and concurrently.
      // But we don't have to care since our calculator will enqueue method calls.
      this.calculator.AddSample( value );
    }
       
    // Details skipped.
}
```

Behind the scenes, each invocation of `AverageCalculator.AddSample` is added to the message queue by the <xref:PostSharp.Patterns.Threading.ActorAttribute>, which then processes each call sequentially in the order it was added to the queue. This gives us the guarantee that an instance of the `AverageCalculator` class is never being accessed concurrently by two threads, and eliminates the need to take multithreading into account. 


## Rules enforced by the Actor aspect

At build time, the Actor aspect emits an error in the following situations:

* The class has async methods that are not annotated with the <xref:PostSharp.Patterns.Threading.ReentrantAttribute> attribute (non-reentrant async methods are not yet supported in actors). 

* The class has public or internal instance fields.

At run time, an actor will throw a <xref:PostSharp.Patterns.Threading.ThreadMismatchException> if some code attempts to access a field from a thread that does not currently have access to the object. This typically happens when you schedule a background task or register an event handler, and you do not mark this method with the <xref:PostSharp.Patterns.Threading.EntryPointAttribute> custom attribute. 


## Working with a complex state

PostSharp generates code that prevents the fields of an actor class from being accessed from an invalid context. For instance, trying to read an actor field from a background task would result in a <xref:PostSharp.Patterns.Threading.ThreadMismatchException>. However, very often, the state is more complex than fields of simple types like `int` or `string`. The state can be composed of several objects and collections. 

To prevent state corruption, it is important that PostSharp generates code that enforces the Actor model at run time even for child objects of the actor.


### Adding complex state to actor classes

1. Declare the parent-child relationship on the property using the <xref:PostSharp.Patterns.Model.ChildAttribute> custom attribute. 


2. Add the <xref:PostSharp.Patterns.Threading.PrivateThreadAwareAttribute> attribute to the child class. 


For more information regarding parent-child relationships in threading models, see <xref:aggregatable>. 


### Example

```csharp
[Actor]          
class AverageCalculator
{
    float sum;
    int count;
    
    [Child]
    private CounterInfo counterInfo;
    
    // Other details skipped for brevity
}

[PrivateThreadAware]
public class CounterInfo
{
    public string Name { get; set; }
}
```


## Dealing with constraints of the Actor model

By definition of the Actor model, all methods are executed asynchronously. Methods that have no return value (void methods) can be executed asynchronously without syntactic changes. However, methods that do have a return value need to be made asynchronous using the `async` keyword. 

In some situations, the application of the `async` keyword and the corresponding dispatching of the method may be unnecessary. For instance, a method that returns immutable information is always thread-safe and does not need to be dispatched. For more information on excluding methods from dispatching, see <xref:threading-waiving-verification>. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Threading.ActorAttribute>
<br><xref:PostSharp.Patterns.Model.ChildAttribute>
<br><xref:PostSharp.Patterns.Model.ParentAttribute>
<br><xref:PostSharp.Patterns.Threading.PrivateThreadAwareAttribute>
<br><xref:PostSharp.Patterns.Model.ReferenceAttribute>
<br><xref:PostSharp.Patterns.Threading.ThreadMismatchException>

**Other Resources**

<xref:threading-waiving-verification>
