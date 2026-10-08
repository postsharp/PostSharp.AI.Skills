---
uid: weak-event
redirect_from:
  - model/weakevent/weak-event
title: "Weak Event"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use the PostSharp.Patterns.Model.WeakEventAttribute aspect to prevent memory leaks in .NET by turning strong event references into weak ones, allowing for garbage collection of event clients."
---
# Weak Event

Event handlers are often a source of memory leaks in .NET. The reason is that the standard .NET event implementation holds a strong reference to the delegates registered as event handlers, which, in turn, hold a strong reference to the event client. As a result, event clients are not garbage collected as long as the event source is alive unless the event client delegate is unregistered from the event. Junior developers are often unaware of this behavior, so this is a frequent cause of memory leaks.

The <xref:PostSharp.Patterns.Model.WeakEventAttribute> aspect solves this problem by changing the implementation of the events to which it is applied. Instead of holding a strong reference to the delegates, a weak event holds only a weak reference, and therefore does not prevent the event client from being garbage collected. The <xref:PostSharp.Patterns.Model.WeakEventAttribute> aspect also ensures that delegates of an instance method are not collected as long as the instance itself is alive. 


## Making an event a weak event


### Turning an event into a weak event

1. Add a reference to the `PostSharp.Patterns.Model` package to your project. 


2. Add a reference to the `PostSharp.Patterns.Model` namespace to your file. 


3. Add the <xref:PostSharp.Patterns.Model.WeakEventAttribute> custom attribute to your event. 


Classes consuming weak events can register events as usual and do not need to take care of unregistering the event handler, unless the event handler needs to be unregistered before the end of the lifetime of the consuming class. It is not necessary to implement <xref:System.IDisposable> to unregister the event handlers because this is automatically done during garbage collection. 


### Example

The following code snippet shows a weak event named `MyEvent` and an event client named `MyEventClient`. Note that the `MyEventClient` does not need to take care of unregistering the `OnMyEvent` event handler. 

```csharp
using System;
using PostSharp.Patterns.Model;

static class MyEventSource
{
   [WeakEvent]
   public static event EventHandler MyEvent;
}

class MyEventClient
{
  
  public MyEventClient()
  {
     MyEventSource.MyEvent += OnMyEvent;
  }

  public void OnMyEvent(object sender, EventArgs e)
  {
      Console.WriteLine("Oops!");
  }

}
```


## Making all events weak events

Because there are very few situations in which you would not want an event to be a weak event, you may want to make all events of a project weak events by default. You can do this using aspect multicasting. See <xref:attribute-multicasting> for details. 

The following code snippet makes all events of the current assembly weak events:

```csharp
[assembly: PostSharp.Patterns.Model.WeakEvent]
```

The following code snippet makes all events of a specific namespace weak events: 

```csharp
[assembly: PostSharp.Patterns.Model.WeakEvent(AttributeTargetTypes="OurCompany.OurApplication.ViewModel.*")]
```


