---
uid: undoredo-callbacks
redirect_from:
  - model/undoredo/undoredo-callbacks
title: "Adding Callbacks on Undo and Redo"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use the IRecordableCallback interface in PostSharp to execute code before or after an Undo or Redo operation."
---
# Adding Callbacks on Undo and Redo

> [!CAUTION]
> Undo/Redo was deprecated in PostSharp 2026.0 and will not be maintained. The API will be removed in the future.

You may run into situations where you will want to execute some code before or after an object is modified by an Undo or Redo operation. This capability is provided through the <xref:PostSharp.Patterns.Recording.IRecordableCallback> interface. 

In the following example, we output a message each time an undo or redo operation executes.

```csharp
[Recordable]
public class Invoice : IRecordableCallback
{
  public void OnReplaying(ReplayKind kind, ReplayContext context)
  {
      if (kind == ReplayKind.Redo) {
           Console.WriteLine("I will now redo a previously undone change to the shipping date.");
      }
  }
  
  public void OnReplayed(ReplayKind kind, ReplayContext context)
  {
      if (kind == ReplayKind.Undo) {
           Console.WriteLine("A change to the shipping date is now undone.");
      }
  }
  
  public DateTime ShippingDate { get; set; }   
}
```

For more information, see the reference documentation for the <xref:PostSharp.Patterns.Recording.IRecordableCallback> interface. 


