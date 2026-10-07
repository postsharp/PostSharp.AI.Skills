---
uid: inotifypropertychanged-conceptual
redirect_from:
  - model/notifypropertychanged/inotifypropertychanged-conceptual
title: "Understanding the NotifyPropertyChanged Aspect"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains the NotifyPropertyChanged Aspect in PostSharp, its implementation, how it analyzes field-property dependencies, its limitations, and how it raises notifications. It also covers dependencies on properties of external objects."
---
# Understanding the NotifyPropertyChanged Aspect

This section describes the principles and algorithm on which the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect is based. It helps developers and architects to understand the behavior and limitations of the aspect. 


## Implementation of the INotifyPropertyChanged interface

The <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect introduces the <xref:System.ComponentModel.INotifyPropertyChanged> interface to the target class unless the target class already implements the interface.


## Instrumentation of fields

Although most implementations of the <xref:System.ComponentModel.INotifyPropertyChanged> interface rely on instrumenting the property setter, this strategy has severe limitations: it is unable to handle *composite properties*, which return a value based on several other fields or properties (for instance a `FullName` property appending `FirstName` and `LastName`). Composite properties have no setter, rendering this strategy unusable. 

Instead, the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect instruments all write operations to fields. It analyzes dependencies between fields and properties and raises a change notification for any property affected by a change in this specific field. 

All methods, and not just property setters, can make a change to a field and therefore cause the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event to be raised. Property setters do not have any specific status in the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> implementation. 


## Analysis of field-property dependencies

To adequately raise the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event, the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect needs to know which properties are affected by a change of a class field. The field-property dependency map is created at build time by analyzing the source code: the analyzer reads the getter of all properties and checks for field references. The map is then serialized inside the assembly and used at run time to raise relevant events when a field has changed. 


### Dependencies on fields of the current object

Consider the following code snippet:

```csharp
[NotifyPropertyChanged]              
class Invoice
{
  private decimal _amount;
  private decimal _tax;
  
  public decimal Amount { get { return this._amount; } set { this._amount = value; } }
  public decimal Tax { get { return this._tax; } set { this._tax = value; } }
  
  public void Set( decimal amount, decimal tax )
  {
     this._amount = amount;
     this._tax = tax;
  }
  
  public decimal Total { get { return this._amount + this._tax; } }
}
```

The result of the analysis for the code snippet above would be the map `{ _amount => ( Amount, Total ), _tax => ( Tax, Total ) }`. Whenever the `_amount` field is changed, the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event will be raised for properties `Amount` and `Total`. 

Automatic properties are processed as handwritten properties; in this case, the implicit backing field is taken into account for the dependency analysis.


### Recursive analysis of the call graph

Field references are not only looked for in the getter, but in any method invoked from the getter, and recursively.

Consider the following code snippet:

```csharp
[NotifyPropertyChanged]              
class Invoice
{
  private decimal _amount;
  private decimal _exchangeRate;
  
  public decimal Amount { get { return this._amount; } set { this._amount = value; } }
  public decimal ExchangeRate { get { return this._exchangeRate; } set { this._exchangeRate = value; } }
  
  private decimal Convert( decimal amount )
  {
    return amount * this.ExchangeRate;
  }
  
  public decimal AmountBase { get { return this.Convert( this.Amount ); } }
  
}
```

In the code snippet above, the analyzer starts from the getter of the `AmountBase` property, follows the call to the `Amount` property getter, then the call to the `Convert` method and recursively follows the `ExchangeRate` property getter. Therefore, the resulting property map remains `{ _amount => ( Amount, AmountBase ), _exchangeRate => ( ExchangeRate, AmountBase ) }`. 


### Dependencies on properties of external objects

The <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect does not just handle dependencies between a property and a field of the same class. It also handles dependencies on properties of properties or properties of fields, and recursively. That is, it supports expressions of the form `_f.P1.P2.P3` where `_f` is a field or property and `P1`, `P2` and `P3` are properties. 

Consider the following code snippet:

```csharp
[NotifyPropertyChanged]              
class InvoiceModel
{
  private decimal _amount;
  private decimal _tax;
  
  public decimal Amount { get { return this._amount; } set { this._amount = value; } }
  public decimal Tax { get { return this._tax; } set { this._tax = value; } }
}

[NotifyPropertyChanged]
class InvoiceViewModel
{
  InvoiceModel _model;  
  
  public InvoiceModel Model { get { return this._model; } }
  
  public decimal Total { get { return this._model.Amount + this.Model.Tax; } }
  
}
```

In the example above, the `InvoiceViewModel.Total` property is dependent on properties `Amount` and `Tax` of the `_model` field. Therefore, changes in the `InvoiceModel._amount` field will trigger a change notification for the `InvoiceModel.Amount` and `InvoiceViewModel.Total` properties. 

The <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect automatically subscribes to the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event of the child object, and unsubscribes whenever the value of the field in the parent object (`_model` in our example) is modified.

The parent object does not unsubscribe upon disposal, because the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect makes no assumption that the <xref:System.IDisposable> interface has been implemented. Instead, the subscription is weak, so the child object does not keep the parent object alive. When the child object implements <xref:System.ComponentModel.INotifyPropertyChanged> or <xref:System.ComponentModel.INotifyPropertyChanging> by hand, the aspect subscribes to its events through an intermediate object that holds a weak reference to the handler of the parent. After the parent object has been garbage-collected, its handlers no longer run, and the intermediate object removes itself the next time the child object raises the event. 

Recursive dependencies to external objects are handled by an auxiliary interface named `INotifyChildPropertyChanged`. This interface is implemented by the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect. It is considered an implementation detail and cannot be implemented manually. Classes that do not implement the `INotifyChildPropertyChanged` interface can only participate as terminal dependencies, that is, they can be leaves but not intermediate nodes. 


## Limitations

The design goal of the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect is to be able to handle the majority of use cases in real-world source code while requiring only an acceptable amount of compilation time. The dependency analysis algorithm imposes several limitations: 

* Calls to virtual methods (other than through the `base` keyword), abstract methods, interface methods or delegates are not supported. 

* Calls to static methods or methods of external classes are not supported unless they are decorated with the <xref:PostSharp.Patterns.Model.PureAttribute> custom attribute, or unless the method is a property getter in a supported dependency chain. 

* Dependencies on properties of method return values are not supported. Only properties of fields or properties are supported.

* Dependencies on properties of variables are not supported if the variable is assigned in a loop (`while`, `for`, ...) or in an exception handling block. 

See <xref:inotifypropertychanged-customization> to learn how to cope with these limitations. 


## Raising notifications

Simplistic implementations of the <xref:System.ComponentModel.INotifyPropertyChanged> interface signal a change notification immediately after a property has been changed. However, this strategy may cause subtle errors in client code. 

Consider the following code:

```csharp
[NotifyPropertyChanged]
class Invoice
{
  public decimal Amount { get; private set; }
  public decimal Tax { get; private set; }
  public decimal Total { get; private set; }

  public void Set( decimal amount, decimal tax )
  {
    /* 1 */ this.Amount = amount;
    /* 2 */ this.Tax = tax;
    /* 3 */ this.Total = amount + tax;
  }

}
```

As a class invariant, the assumption `Total == Amount + Tax` should always be true. 

However, suppose that the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event is raised immediately after the `Amount` property is set at line 1 of the `Set` method. Clearly, for a client subscribing to this event, the class invariant would be broken at this specific moment. 

Therefore, it is not safe to raise change notifications immediately after a change has been achieved. It is necessary to wait until the object can be safely observed by external code, when all class invariants are valid again (that is, when the object state is consistent). A common best practice in object-oriented programming is to ensure that class invariants are valid before the control flow goes back from the current object to the caller. Typically, it means that a private or protected method can exit with an inconsistent object state, but public and internal methods must guarantee that the object state is consistent upon exit.

The <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect relies on this best practice and raises the property change notifications just before the control flow exits the current object, that is, just before the last public or internal method in the call stack for the current object exits. 

Besides avoiding the exposure of an invalid object state, this strategy also prevents the same property from being notified several times during the execution of a single public method, which has a potentially large positive impact on performance.

To solve this problem, the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect uses the following strategy: 

* Instead of causing immediate change notifications, field changes are buffered into a thread-local storage named the *accumulator*. 

* Calls to public and internal methods are instrumented so the aspect can detect when the control flow exits the object. At this moment, the accumulator is flushed and all change notifications are triggered.

It is possible to flush the accumulator at any time by invoking the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedServices.RaiseEventsImmediate(System.Object)> method. 

You can suspend and resume notifications using the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedServices.SuspendEvents> and <xref:PostSharp.Patterns.Model.NotifyPropertyChangedServices.ResumeEvents> methods. Each call to <xref:PostSharp.Patterns.Model.NotifyPropertyChangedServices.ResumeEvents> must match a previous call to <xref:PostSharp.Patterns.Model.NotifyPropertyChangedServices.SuspendEvents> on the same thread. If the current thread has not suspended the events, `ResumeEvents` throws an <xref:System.InvalidOperationException>. 


## Remarks

The <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect never evaluates property getters at run time. This decision is deliberate and aims at avoiding possible side effects (lazy initialization, logging, etc.). Therefore, it is possible that the algorithms emit false positives, that is, change notifications for properties whose values did not actually change. 

The algorithm heuristically detects dependency cycles. If a cycle is detected, an exception is thrown instead of allowing for an infinite update cycle.

All notifications are invoked on the thread on which the change is being made. The accumulator that buffers the changes is a thread-local storage.


