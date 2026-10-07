---
uid: threading-waiving-verification
redirect_from:
  - threading/threading-waiving-verification
title: "Opting In and Out From Thread Safety"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to opt in and out of thread safety in PostSharp by using the ExplicitlySynchronizedAttribute and EntryPointAttribute on methods and fields. It also warns about potential multithreading defects."
---
# Opting In and Out From Thread Safety

By default, PostSharp enforces thread safety for all instance fields and all public and internal methods of any class to which you applied a threading model.

However, there are times when you want to opt out of this mechanism for a specific field or method. A typical reason is that access to the field is synchronized manually using a different mechanism.

This section shows how to override the default thread safety implemented by PostSharp.


## Opting out from thread-safety verification for a method

To disable enforcement of the class-level threading model for a specific method, add the <xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute> attribute to that method. 

In the following example, this custom attribute allows us to implement the `ToString` in a class that respects the Actor model. Without the custom attribute, this would not have been possible because non-void public methods must have the `async` keyword. 

```csharp
[Actor]
class Player          
{
   private readonly string name;

   [ExplicitlySynchronized] 
   public override string ToString()
   {
      return this.name;
   }
}
```

When used on a method, the <xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute> attribute has several effects: 

* Lock-based aspects such as <xref:PostSharp.Patterns.Threading.SynchronizedAttribute> or <xref:PostSharp.Patterns.Threading.ReaderWriterSynchronizedAttribute> will not attempt to acquire a lock before executing this method. 

* Accesses to fields are not verified during the whole execution of the method (for the current thread).

* All build-time verifications are disabled for this method.

> [!WARNING]
> By using the <xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute> custom attribute, you are significantly increasing the risk that multithreading defects in user code go undetected by PostSharp. Code using <xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute> should be more carefully covered by reviews and tests. 


## Opting out from thread-safety for a field

To disable enforcement of the class-level threading model for a specific field, add the <xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute> attribute to the field: 

```csharp
[Actor]
class MyActor
{

  [ExplicitlySynchronized]
  int counter;

  public void FooBar()
  {
    // This line would throw an exception without [ExplicitlySynchronized].
    Task.Factory.StartNew(() => Interlocked.Increment( ref this.counter ));
  }
  
}
```

When used on a field, the <xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute> attribute has several effects: 

* Accesses to the field are never verified.

* All build-time verifications are disabled for this field.


## Opting in for thread safety for callback methods

By default, thread safety is ensured when a thread first invokes a public or internal method of an object. The underlying motivation is that public and internal methods are the primary way in which a thread can enter an object. Another way is to enter an object through a delegate call to a private method. By default, PostSharp does not ensure thread safety for private methods. If you register a callback method, you need to add the <xref:PostSharp.Patterns.Threading.EntryPointAttribute> custom attribute on this method. 

In the following code snippet, the `OnCreated` method is invoked from a background thread by the `FileSystemWatcher` class. The `InputQueueWatcher` is thread-safe because of the <xref:PostSharp.Patterns.Threading.SynchronizedAttribute> aspect. 

```csharp
[Synchronized]
class InputQueueWatcher         
{
  FileSystemWatcher watcher;
  
  [Child]
  AdvisableCollection<string> files = new AdvisableCollection<string>();
  
  public InputQueueWatcher(string path)
  {
    this.watcher = new FileSystemWatcher();
    this.watcher.Path = path;
    this.watcher.NotifyFilter = NotifyFilters.LastWrite | NotifyFilters.FileName | NotifyFilters.DirectoryName;
    this.watcher.Filter = "*.xml";
    this.watcher.Created += new FileSystemEventHandler(OnCreated);
  }
  
  [EntryPoint]
  private void OnCreated(object source, FileSystemEventArgs e)
  {
     // Without [EntryPoint], the following line would throw ThreadAccessException.
     this.files.Add(e.FullPath);
  }
   
   public ICollection Files { get { return this.files; } }
   
 }
```

## See Also

**Reference**

<xref:PostSharp.Patterns.Threading.ExplicitlySynchronizedAttribute>
<br><xref:PostSharp.Patterns.Threading.EntryPointAttribute>

**Other Resources**

<xref:thread-unsafe>
<br><xref:reader-writer-synchronized>
<br><xref:ui-dispatching>
<br><xref:actor>
