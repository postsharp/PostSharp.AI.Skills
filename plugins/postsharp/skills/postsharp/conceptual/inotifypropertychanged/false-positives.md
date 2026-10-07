---
uid: inotifypropertychanged-false-positives
redirect_from:
  - model/notifypropertychanged/inotifypropertychanged-false-positives
title: "Suppressing False Positives of the NotifyPropertyChanged Aspect"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to suppress false positives in the NotifyPropertyChanged Aspect in PostSharp to avoid redundant event notifications, thereby improving application responsiveness. However, it warns against suppressing false positives when property getters have side effects."
---
# Suppressing False Positives of the NotifyPropertyChanged Aspect

The <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect, when applied to a class, raises the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event every time it detects a possible change of a property, even when the actual value of the property does not change. By default, the aspect does not keep track of the property values because that would require the aspect to invoke property getters arbitrarily outside of the developer’s control. And when property getters have any side effects (lazy-initialization, logging, etc.), invoking them randomly is not a safe behavior. 

In certain scenarios, such as rich client applications with many UI controls, redundant event notifications are not desired because they cause excessive UI updates and can degrade the application responsiveness. You can avoid these redundant event notifications by suppressing false positives in the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect. 


## Example of a false positive

The following `Calc` class has two integer fields `a` and `b`, and a property `Sum` that returns the sum of these two numbers. The `Main` method creates an instance of the `Calc` class and changes the fields from `(a=1, b=2)` to `(a=2, b=1)`. There are two `PropertyChanged` event notifications shown in the output, even though the actual `Sum` value does not change in the second case. 

```csharp
[NotifyPropertyChanged]
class Calc
{
    private int a, b;

    public int Sum
    {
        get { return this.a + this.b; }
    }

    public void Update(int a1, int b1)
    {
        this.a = a1;
        this.b = b1;
    }
}

static void Main()
{
    Calc calc = new Calc();
    ((INotifyChildPropertyChanged) calc).PropertyChanged +=
        (sender, eventArgs) =>
        {
            Console.WriteLine("Property {0} changed. New value = {1}.",
                eventArgs.PropertyName,
                sender.GetType().GetProperty(eventArgs.PropertyName).GetValue(sender));
        };

    calc.Update(1, 2);
    calc.Update(2, 1);
}
```

Output:

```none
Property Sum changed. New value = 3.
Property Sum changed. New value = 3.
```


## How to suppress false positives

To suppress false positive event notifications, set the `PreventFalsePositives` property to `true` when you apply the aspect to the target element.

```csharp
[NotifyPropertyChanged(PreventFalsePositives = true)]
class Calc
{
    // ...
}
```

If you run the test code snippet again after this change, you can see that there is only one change notification now.

```none
Property Sum changed. New value = 3.
```

> [!WARNING]
> Do not suppress false positive notifications in your class when the property getters in the class have side effects. In this case, reset the `PreventFalsePositives` property to its default value of `false`. 

By enabling the `PreventFalsePositives` option of the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect you can reduce the number of events raised in your application and improve your UI responsiveness. 

## See Also

**Reference**

<xref:System.ComponentModel.INotifyPropertyChanged>
<br><xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>

**Other Resources**

<xref:inotifypropertychanged-conceptual>
