---
uid: undoredo-start
redirect_from:
  - model/undoredo/undoredo-start
title: "Making Your Model Recordable"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on making an object recordable for undo/redo operations using PostSharp's RecordableAttribute aspect, and how to exclude a property from these operations."
---
# Making Your Model Recordable

> [!CAUTION]
> Undo/Redo was deprecated in PostSharp 2026.0 and will not be maintained. The API will be removed in the future.

To make an object usable for undo/redo operations, you will need to add the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect to the class. This aspect instruments changes to fields and records them into a <xref:PostSharp.Patterns.Recording.Recorder>. The aspect also instruments public methods to group field changes into logical operations. 


## Making the class recordable

1. Add the `PostSharp.Patterns.Model` package to the project. 


2. Add the <xref:PostSharp.Patterns.Recording.RecordableAttribute> to the class. 


3. Annotate your object model for parent/child relationships as explained in <xref:aggregatable-adding>. 



## Example

The following example shows an object model that has been prepared for undo/redo.

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoModel.cs#example)]

By adding the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect to the classes, all modifications to properties of these classes (including modifications to child collections) will be recorded. 


## Excluding a property from undo/redo

To exclude a property from participating in undo/redo, annotate that property with <xref:PostSharp.Patterns.Recording.NotRecordedAttribute>. 

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoNotRecorded.cs#example)]

After an undo, the `Text` property gets its previous value, but the `AnyChangesMade` property keeps its value:

[!code-text[](../../../code/samples/Model/UndoRedo/UndoRedoNotRecorded.txt)]


