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

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoCallbacks.cs#example)]

When a program changes the shipping date, then calls <xref:PostSharp.Patterns.Recording.Recorder.Undo> and <xref:PostSharp.Patterns.Recording.Recorder.Redo>, the output is:

[!code-text[](../../../code/samples/Model/UndoRedo/UndoRedoCallbacks.txt)]

For more information, see the reference documentation for the <xref:PostSharp.Patterns.Recording.IRecordableCallback> interface. 


