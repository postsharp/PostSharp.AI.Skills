---
uid: reader-writer-synchronized
redirect_from:
  - threading/reader-writer-synchronized
title: "Reader/Writer Synchronized Threading Model"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document details the Reader/Writer Synchronized Threading Model in PostSharp, which uses locks to prevent data races in multithreaded programs. It explains how to implement the model and provides usage examples."
---
# Reader/Writer Synchronized Threading Model

When a class instance is concurrently used by multiple threads, accesses must be synchronized to prevent data races, which typically result in data inconsistencies and corruption of data structures. The Reader/Writer Synchronized Threading Model uses locks to allow several read-only methods to execute simultaneously on one instance, but guarantee that writer methods have exclusive access.

The Reader/Writer Synchronized Threading Model is implemented by the <xref:PostSharp.Patterns.Threading.ReaderWriterSynchronizedAttribute> aspect. It requires you to annotate all public methods of your synchronized classes with the <xref:PostSharp.Patterns.Threading.ReaderAttribute> and <xref:PostSharp.Patterns.Threading.WriterAttribute> custom attributes. 


## Why use the reader-writer synchronized pattern?


### Problems without locks

Consider the following example of an `Order` class which stores an amount and a discount: 

```csharp
class Order  
{        
    int Amount { get; private set; }
    int Discount { get; private set; }

    public int AmountAfterDiscount
    {
        get { return this.Amount - this.Discount; }
    }
     
    public void Set(int amount, int discount)
    {
        if (amount < discount)
            throw new InvalidOperationException();

        this.Amount = amount;
        this.Discount = discount;
    }
}
```

In this example, the `Set` method writes to the `Amount` and `Discount` members, while the `AmountAfterDiscount` property reads these members. In a single-threaded program, the `AmountAfterDiscount` property is guaranteed to be positive or zero. However, in a multithreaded program, the `AmountAfterDiscount` property could be evaluated in the middle of the `Set` operation, and return an inconsistent result. 


### Problems of the `lock` keyword

The easiest way to synchronize accesses to a class in C# is to use the `lock` keyword. However, this practice cannot be generalized for two reasons:

* The use of exclusive locks often results in high contention and therefore low performance because many threads queue to access the same resource;

* Applications relying on exclusive locks are prone to deadlocks because of cyclic waiting dependencies.


### Problems of reader-writer locks

Reader-writer locks take advantage of the fact that most applications involve far fewer writes than reads, and that concurrent reads are always safe. Reader-writer locks ensure that no other thread is accessing the object when it is being written. Reader-writer locks are normally implemented by the .NET classes <xref:System.Threading.ReaderWriterLock> or <xref:System.Threading.ReaderWriterLockSlim>. The following example shows how <xref:System.Threading.ReaderWriterLockSlim> would be used to control reads and writes in the `Order` class: 

```csharp
class Order
{
    private ReaderWriterLockSlim orderLock = new ReaderWriterLockSlim();

    public decimal Amount { get; private set; }
    public decimal Discount { get; private set; }

    public decimal AmountAfterDiscount
    {
        get 
        {
            orderLock.EnterReadLock();
            decimal result = this.Amount - this.Discount;
            orderLock.ExitReadLock();
            return result;
        }
    }

    public void Set(decimal amount, decimal discount)
    {

        if (amount < discount)
        {
            throw new InvalidOperationException();
        }

        orderLock.EnterWriteLock();
        this.Amount = amount;
        this.Discount = discount;
        orderLock.ExitWriteLock();
    }    
}
```

However, working directly with the <xref:System.Threading.ReaderWriterLock> and <xref:System.Threading.ReaderWriterLockSlim> classes has disadvantages: 

* It is cumbersome because a lot of code is required.

* It is unreliable because it is too easy to forget to acquire the right type of lock, and these errors are not detectable by the compiler or by unit tests.

So, the direct use of locks not only results in more lines of code, but also does not reliably prevent nondeterministic data structure corruption.


## Making a class reader-writer synchronized

PostSharp Threading Pattern Library has been designed to eliminate nondeterministic data corruptions while reducing the size of thread synchronization code to the absolute minimum (but not less).

The <xref:PostSharp.Patterns.Threading.ReaderWriterSynchronizedAttribute> aspect implements the threading model (or threading pattern) based on the reader-writer lock, with the following principles: 

* At any time, the object can be open for reading or closed for reading.

* Methods define their required access level using <xref:PostSharp.Patterns.Threading.ReaderAttribute> and <xref:PostSharp.Patterns.Threading.WriterAttribute> custom attributes (other access levels exist for advanced scenarios). 

* An error will be emitted at build time or run time, but deterministically, whenever an object field is being accessed by a method that does not have the required access level on the object.


To apply the ReaderWriterSynchronized threading model to a class:

1. Add the `PostSharp.Patterns.Threading` package to your project. 


2. Import the `PostSharp.Patterns.Threading` namespace into your file. 


3. Add the custom attribute [<xref:PostSharp.Patterns.Threading.ReaderWriterSynchronizedAttribute>] to the class. 


4. Annotate your object model for parent/child relationships as described in <xref:aggregatable-adding>. 


5. Add the custom attribute [<xref:PostSharp.Patterns.Threading.ReaderAttribute>] or [<xref:PostSharp.Patterns.Threading.WriterAttribute>] to the public and internal methods. Note that it is not necessary to put these attributes on property getters and setters or on events. 


The <xref:PostSharp.Patterns.Threading.ReaderAttribute> attribute causes PostSharp to acquire a lock on the instance whenever the method is invoked. While this lock is held, other threads can also read properties or invoke read-only methods of that instance, but calls to properties or methods marked with <xref:PostSharp.Patterns.Threading.WriterAttribute> will be blocked until all reads are complete. 

Likewise, invoking methods marked with <xref:PostSharp.Patterns.Threading.WriterAttribute> will lock the instance causing all reads and writes to block until the write has completed and the write lock has been released. 


### Example

The following code shows the `Order` class, synchronized with the reader-writer threading pattern: 

```csharp
[ReaderWriterSynchronized]
class Order
{        
    decimal Amount { get; private set; }
    decimal Discount { get; private set; }

    public decimal AmountAfterDiscount
    {
        get { return this.Amount - this.Discount; }
    }
    
    [Writer]
    public void Set(decimal amount, decimal discount)
    {
        if (amount < discount)
            throw new InvalidOperationException();

        this.Amount = amount;
        this.Discount = discount;
    }    
}
```


## Rules enforced by the ReaderWriterSynchronized aspect

The <xref:PostSharp.Patterns.Threading.ReaderWriterSynchronizedAttribute> aspect emits build-time errors in the following situations: 

* The class contains a public or internal field.

The aspect emits a build-time warning when the class contains a public or internal method that is missing a <xref:PostSharp.Patterns.Threading.ReaderAttribute> or <xref:PostSharp.Patterns.Threading.WriterAttribute> custom attribute, or another attribute derived from <xref:PostSharp.Patterns.Threading.AccessLevelAttribute>. Note that property getters and setters and event accessors do not need to be annotated. 

The reader/writer synchronized object will throw a <xref:PostSharp.Patterns.Threading.ThreadAccessException> whenever some code tries to access a field from a thread that does not own the correct lock, that is, when a reader method tries to write a field, or when a non-annotated method (for example, a delegate call) tries to read or write a field. 


## Raising synchronous events

In some situations, a method with write access needs to allow other threads to read the object before another write is performed on the object. The implementation of <xref:System.Collections.Specialized.INotifyCollectionChanged> gives a typical example of this situation. The <xref:System.Collections.Specialized.INotifyCollectionChanged> event defined by this interface is typically raised from a write method but is consumed from the user interface thread. The object cannot have changed between the moment the event is raised and the moment it is processed by the UI thread, because the event arguments contain data that relates to the current state of the object. Using only <xref:PostSharp.Patterns.Threading.WriterAttribute> and <xref:PostSharp.Patterns.Threading.ReaderAttribute> would either result in deadlocks or in inconsistencies, respectively. 

The solution to this problem is to use the <xref:PostSharp.Patterns.Threading.YielderAttribute> custom attribute, which allows read access from other threads but prevents any other thread from acquiring a writer lock. 

When a thread that has no access to the object calls a method annotated with <xref:PostSharp.Patterns.Threading.YielderAttribute>, the method acquires read access, which other threads share. Read access cannot be upgraded. If a handler of the event calls a method that requires write access on the same thread, for instance a method annotated with <xref:PostSharp.Patterns.Threading.WriterAttribute>, this method throws a <xref:System.Threading.LockRecursionException>.

When a handler requires write access, annotate the method that raises the event with <xref:PostSharp.Patterns.Threading.UpgradeableReaderAttribute> instead. Upgradeable read access is exclusive: while the method runs, no other thread can acquire upgradeable read access or write access.


### Example

In the following example, `OrderCollection` is a collection of `Order` objects. In this example, the `Add()` and `Remove()` methods are marked with the <xref:PostSharp.Patterns.Threading.WriterAttribute> attribute. Listeners can be notified about these changes by subscribing to the <xref:System.Collections.Specialized.INotifyCollectionChanged.CollectionChanged> event which is exposed through the implementation of <xref:System.Collections.Specialized.INotifyCollectionChanged>. 

Since listeners can be on other threads (for example, a UI thread), this event is invoked by the `Add()` and `Remove()` methods via a method called `OnCollectionChanged()` which has been marked with the <xref:PostSharp.Patterns.Threading.YielderAttribute> attribute. This lock ensures that the listener (which may be in another thread space) can read the current state of the collection without the collection being modified by another invocation of the `Add()` or `Remove()` operations from another thread. 

```csharp
[ReaderWriterSynchronized]
class OrderCollection : ICollection, INotifyCollectionChanged
{
    ArrayList list = new ArrayList();

    // Details skipped.

    public int Count
    {
        [Reader]
        get
        {
            return list.Count;
        }
    }

    [Writer]
    public void Add(Order o)
    {
        list.Add(o);
        NotifyCollectionChangedEventArgs changedArgs = new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Add, o);
        OnCollectionChanged(changedArgs);
    }

    [Writer]
    public void Remove(int index)
    {
        NotifyCollectionChangedEventArgs changedArgs = new NotifyCollectionChangedEventArgs(NotifyCollectionChangedAction.Remove, list[index]);            
        list.RemoveAt(index);
        OnCollectionChanged(changedArgs);
    }

    [Yielder]
    private void OnCollectionChanged(NotifyCollectionChangedEventArgs changedArgs)
    {
        CollectionChanged(this, changedArgs);
    }

    [Reader]
    public Order Get(int index)
    {
        return (Order)list[index];
    }

    public event NotifyCollectionChangedEventHandler CollectionChanged;
}
```


## Executing long-running write methods

Since write methods require exclusive access to the object, they should complete as quickly as possible. However, this is not always possible. Some long-running write methods really do a lot of write operations (or rely on slow external services) which make them inappropriate for the reader-writer-synchronized model. However, many write methods are actually composed of a lot of read operations but only a few write operations at the end. In this case, it is possible to use a combination of the <xref:PostSharp.Patterns.Threading.UpgradeableReaderAttribute> and <xref:PostSharp.Patterns.Threading.WriterAttribute> attributes. The <xref:PostSharp.Patterns.Threading.UpgradeableReaderAttribute> attribute ensures that no thread other than the current one will be able to acquire a writer lock on the object, so it gives the guarantee that the object is not going to be modified during the method’s execution. A method that holds an upgradeable reader lock can then invoke a method with the <xref:PostSharp.Patterns.Threading.WriterAttribute> custom attribute. Note that it is important that the writer methods leave the object in a consistent state before exiting, because other threads will be allowed to read the object. 

In the following example, the `Order` class contains a collection of `Line` objects which make up the order. In the example below, a new method called `Recalculate()` has been added to `Order` which iterates through each `Line` in the collection, tallies up the amount from each, and then stores the total in `Amount`. 

Since the `Recalculate` method performs a series of reads followed by a write operation (to store the total in `Amount`), it is marked with the <xref:PostSharp.Patterns.Threading.UpgradeableReaderAttribute> attribute which ensures that all of the orders that it reads remain locked so that it calculates and writes out the correct total. In addition to this, the set accessor of the `Order`’s `Amount` property has been marked with <xref:PostSharp.Patterns.Threading.WriterAttribute>: 

```csharp
[ReaderWriterSynchronized]
class Order
{
    // Other details skipped for brevity.

    public decimal Amount
    {
       // The [Reader] attribute is optional here because the method is a public getter.
       get;
       
       // The [Writer] attribute is required because, although the method is a setter, this setter is private, 
       // therefore it does not acquire write access by default.
       [Writer] private set;
    }

    [UpgradeableReader]
    public void Recalculate()
    {
        decimal total = 0;
        
        for (int i = 0; i < lines.Count; ++i)
        {
            total += lines[i].Amount;
        }

        this.Amount = total;
    }
}
```


## Working with object trees

Because the Reader/Writer Synchronized model is an implementation of the Aggregatable pattern, all of the same behaviors of the <xref:PostSharp.Patterns.Model.AggregatableAttribute> are available. For more information regarding object trees, see <xref:aggregatable>. 

> [!NOTE]
> Once you have established your parent-child relationships, you will need to apply compatible threading models to the child classes. You will want to refer to the <xref:threading-model-compatibility> article to determine which threading model will work for the children of the Reader/Writer Synchronized object. 

When a ReaderWriterSynchronized object becomes the child of a Synchronized object, it effectively becomes fully Synchronized itself. From that point on, even its Reader methods will require the full lock and will act as though they were Writer methods. When it stops being a child of a Synchronized object, you will again be able to run multiple Reader methods at the same time.

## See Also

**Other Resources**

<xref:threading-model-compatibility>

**Reference**

<xref:PostSharp.Patterns.Threading.ReaderWriterSynchronizedAttribute>
<br><xref:PostSharp.Patterns.Threading.WriterAttribute>
<br><xref:PostSharp.Patterns.Threading.ReaderAttribute>
<br><xref:PostSharp.Patterns.Threading.UpgradeableReaderAttribute>
<br><xref:PostSharp.Patterns.Threading.YielderAttribute>
<br><xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute>
