---
uid: event-interception
redirect_from:
  - custompatterns/aspects/tutorials/event-interception
title: "Intercepting Events"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use PostSharp to intercept events such as subscribing, unsubscribing, and raising. It provides detailed steps for intercepting add and remove actions, raise actions, and accessing the current context."
---
# Intercepting Events

You interact with events in three primary ways: subscribing, unsubscribing and raising them. Like methods and properties, you may find yourself needing to intercept these three interactions. How do you execute code every time that an event is subscribed to? Or raised? Or unsubscribed? PostSharp provides a simple mechanism to accomplish this.


## Intercepting add and remove

Throughout the life of an event, it is possible to have many different event handlers subscribe and unsubscribe. You may want to log each of these actions.

1. Add a reference to the *PostSharp* package to your project. 


2. Create an aspect that inherits from <xref:PostSharp.Aspects.EventInterceptionAspect>. Add the [<xref:PostSharp.Serialization.PSerializableAttribute>] custom attribute. 


3. Override the <xref:PostSharp.Aspects.EventInterceptionAspect.OnAddHandler(PostSharp.Aspects.EventInterceptionArgs)> method and add your logging code to the method body. 


4. Add the `base.OnAddHandler` call to the body of the <xref:PostSharp.Aspects.EventInterceptionAspect.OnAddHandler(PostSharp.Aspects.EventInterceptionArgs)> method. If this is omitted, the original call to add a handler will not be executed. Unless you want to stop the addition of the handler, add this line of code. 

    ```csharp
    [PSerializable]
    public class CustomEventHandling : EventInterceptionAspect
    {
        public override void OnAddHandler(EventInterceptionArgs args)
        {
            base.OnAddHandler(args);
            Console.WriteLine("A handler was added");
        }
    }
    ```


5. To log the removal of an event handler, override the <xref:PostSharp.Aspects.EventInterceptionAspect.OnRemoveHandler(PostSharp.Aspects.EventInterceptionArgs)> method. 


6. Add the logging you require to the method body.


7. Add the `base.OnRemoveHandler` call to the body of the <xref:PostSharp.Aspects.EventInterceptionAspect.OnRemoveHandler(PostSharp.Aspects.EventInterceptionArgs)> method. Like you saw when overriding the <xref:PostSharp.Aspects.EventInterceptionAspect.OnAddHandler(PostSharp.Aspects.EventInterceptionArgs)> method, if you omit this call, the original call to remove the handler will not occur. 

    ```csharp
    public override void OnRemoveHandler(EventInterceptionArgs args) 
    { 
        base.OnRemoveHandler(args); 
        Console.WriteLine("A handler was removed"); 
    }
    ```


Once you have defined the interception points in the aspect, you will need to attach the aspect to the target code. The simplest way to do this is to add the attribute to the event declaration.

```csharp
public class Example 
{
    [CustomEventHandling]
    public event EventHandler<EventArgs> SomeEvent; 
 
    public void DoSomething() 
    { 
        if (SomeEvent != null) 
        { 
            SomeEvent.Invoke(this, EventArgs.Empty); 
        } 
    } 
}
```


## Intercepting raise

When you are intercepting events, you may also have situations where you want to execute additional code when the event is raised. Raising of an event can occur in many places, and you will want to centralize this code to avoid repetition.


1. Override the <xref:PostSharp.Aspects.EventInterceptionAspect.OnInvokeHandler(PostSharp.Aspects.EventInterceptionArgs)> method in your aspect class and add the logging you require to the method body. 


2. Add a call to `base.OnInvokeHandler` to ensure that the original invocation occurs. 

    ```csharp
    public override void OnInvokeHandler(EventInterceptionArgs args) 
    { 
        base.OnInvokeHandler(args); 
        Console.WriteLine("A handler was invoked"); 
    }
    ```


By adding the attribute to the target event earlier in this process, you have enabled interception of each raised event.


## Accessing the current context

At any time, the <xref:PostSharp.Aspects.EventInterceptionArgs.Handler> property is set to the delegate being added, removed, or invoked. This property is read-only. To add, remove, or invoke a different delegate, call the <xref:PostSharp.Aspects.EventInterceptionArgs.AddHandler(System.Delegate)>, <xref:PostSharp.Aspects.EventInterceptionArgs.RemoveHandler(System.Delegate)>, or <xref:PostSharp.Aspects.EventInterceptionArgs.InvokeHandler(System.Delegate,PostSharp.Aspects.Arguments)> method. The delegate you pass must be compatible with the type of the event. The <xref:PostSharp.Aspects.EventInterceptionArgs.Event> property gives you the <xref:System.Reflection.EventInfo> of the event being accessed. 

Within <xref:PostSharp.Aspects.EventInterceptionAspect.OnInvokeHandler(PostSharp.Aspects.EventInterceptionArgs)>, the property <xref:PostSharp.Aspects.EventInterceptionArgs.Arguments> gives access to the arguments with which the delegate was invoked. 

These concepts will be illustrated in the following example.


## Example: Removing offending event subscribers

When events are subscribed to, the component that raises the event has no way to ensure that the subscriber will behave properly when that event is raised. The subscribing code may throw an exception when the event is raised. When that happens, you may want to unsubscribe the handler to ensure that it does not continue to throw the exception. The <xref:PostSharp.Aspects.EventInterceptionAspect> can help you to accomplish this. 


1. Override the <xref:PostSharp.Aspects.EventInterceptionAspect.OnInvokeHandler(PostSharp.Aspects.EventInterceptionArgs)> method in your aspect. 


2. In the method body, add a `try...catch` block. 


3. In the `try` block, add a call to `base.OnInvokeHandler`, and in the `catch` block, add a call to <xref:PostSharp.Aspects.EventInterceptionArgs.RemoveHandler(System.Delegate)>. 

    ```csharp
    [PSerializable]
    public class CustomEventHandling : EventInterceptionAspect 
    { 
        public override void OnInvokeHandler(EventInterceptionArgs args) 
        { 
            try 
            { 
                base.OnInvokeHandler(args); 
            } 
            catch (Exception e) 
            { 
                Console.WriteLine("Handler '{0}' invoked with arguments {1} failed with exception {2}.", 
                                  args.Handler.Method,  
                                  string.Join(", ", args.Arguments.Select(a => a == null ? "null" : a.ToString())), 
                                  e.GetType().Name); 
     
                args.RemoveHandler(args.Handler);  
                throw; 
            } 
        } 
     
    }
    ```


Now, any time an exception is thrown during event execution, the offending event handler will be unsubscribed from the event.


## Intercepting the event initializer

The <xref:PostSharp.Aspects.EventInterceptionAspect.OnAddHandler(PostSharp.Aspects.EventInterceptionArgs)> method does not intercept the initializer of a field-like event. If you want to intercept the adding of all handlers, do not use event initializers and instead add the initial handler in the constructor. 

```csharp
public class TargetClass 
{ 
    [EventInterception]
    public event EventHandler FieldLikeEvent = EventHandler1; // will not be intercepted
    public TargetClass() 
    {
        this.FieldLikeEvent += EventHandler2; // will be intercepted
    } 
}
```

## See Also

**Reference**

<xref:PostSharp.Aspects.EventInterceptionAspect>
<br><xref:PostSharp.Aspects.EventInterceptionArgs.Handler>
<br><xref:PostSharp.Aspects.EventInterceptionArgs.Event>
<br><xref:PostSharp.Serialization.PSerializableAttribute>
<br><xref:PostSharp.Aspects.EventInterceptionAspect.OnAddHandler(PostSharp.Aspects.EventInterceptionArgs)>
<br><xref:PostSharp.Aspects.EventInterceptionAspect.OnRemoveHandler(PostSharp.Aspects.EventInterceptionArgs)>
<br><xref:PostSharp.Aspects.EventInterceptionAspect.OnInvokeHandler(PostSharp.Aspects.EventInterceptionArgs)>
<br><xref:System.Reflection.EventInfo>
<br><xref:PostSharp.Aspects.EventInterceptionArgs.RemoveHandler(System.Delegate)>
