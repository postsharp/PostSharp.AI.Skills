---
uid: undoredo-conceptual
redirect_from:
  - model/undoredo/undoredo-conceptual
title: "Understanding the Recordable Aspect"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains the Recordable Aspect in PostSharp, which records changes performed on instances of a class. It covers topics like scopes, logical operations, atomic operations, primitive operations, restore points, implementing IEditableObject, callback methods, and memory consumption."
---
# Understanding the Recordable Aspect

> [!CAUTION]
> Undo/Redo was deprecated in PostSharp 2026.0 and will not be maintained. The API will be removed in the future.

This section describes how the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect is implemented. It helps developers and architects to understand the behavior and limitations of the aspect. 


## Overview

When the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect is applied to a class, the aspect records changes performed on instances of this class. Changes are represented as instances of the <xref:PostSharp.Patterns.Recording.Operation> class. For instance, the <xref:PostSharp.Patterns.Recording.Operations.FieldOperation`1> class represents the operation of changing the value of a field. All operations implement the <xref:PostSharp.Patterns.Recording.Operation.Undo(PostSharp.Patterns.Recording.ReplayContext)> and <xref:PostSharp.Patterns.Recording.Operation.Redo(PostSharp.Patterns.Recording.ReplayContext)> methods. For instance, the <xref:PostSharp.Patterns.Recording.Operations.FieldOperation`1> class stores both the new and old value so that the operation can be undone and redone. 

The changes are recorded into the <xref:PostSharp.Patterns.Recording.Recorder> object. The <xref:PostSharp.Patterns.Recording.Recorder> maintains two collections of operations: <xref:PostSharp.Patterns.Recording.Recorder.UndoOperations> and <xref:PostSharp.Patterns.Recording.Recorder.RedoOperations>. The <xref:PostSharp.Patterns.Recording.Recorder.Undo> method takes the last operation from the <xref:PostSharp.Patterns.Recording.Recorder.UndoOperations> collection, invokes <xref:PostSharp.Patterns.Recording.Operation.Undo(PostSharp.Patterns.Recording.ReplayContext)> for this operation, and moves the operation to the <xref:PostSharp.Patterns.Recording.Recorder.RedoOperations> collection. The <xref:PostSharp.Patterns.Recording.Recorder.Redo> method works symmetrically. 

It would not be safe, however, to allow users to undo changes in the object model back to any arbitrary point in history. Users do not want to undo primitive changes to an object model, but to undo whole operations understood from a user's perspective. This is why the <xref:PostSharp.Patterns.Recording.Recorder.UndoOperations> and <xref:PostSharp.Patterns.Recording.Recorder.RedoOperations> collections do not expose primitive changes on the object model but logical operations. 

By default, logical operations are automatically opened when calling a public or internal method of a recordable object, and closed when the same method exits. The principal use case of scopes is to define user-friendly operation names.

There is typically a single instance of the <xref:PostSharp.Patterns.Recording.Recorder> class per application, but there could be many if needed (for instance in a multi-document application). The default single instance is accessible from the <xref:PostSharp.Patterns.Recording.RecordingServices.DefaultRecorder> property. By default, recordable objects are attached to the default recorder immediately after completion of the constructor. See <xref:undoredo-recorder> to learn how to customize this behavior. 


## Scopes and Logical Operations

Scopes are a mechanism to aggregate several primitive operations into logical operations that make sense for the end user. Logical operations are represented by the <xref:PostSharp.Patterns.Recording.Operations.CompositeOperation> class. 

In general, logical operations form a flat structure: the <xref:PostSharp.Patterns.Recording.Recorder.UndoOperations> and <xref:PostSharp.Patterns.Recording.Recorder.RedoOperations> collections are flat doubly linked lists, and each <xref:PostSharp.Patterns.Recording.Operations.CompositeOperation> typically contains primitive operations such as a field value change. 

Scopes define boundaries of logical operations. Scopes can be opened using the <xref:PostSharp.Patterns.Recording.Recorder.OpenScope(PostSharp.Patterns.Recording.RecordingScopeOption)> method, which returns an object of type <xref:PostSharp.Patterns.Recording.RecordingScope>. This class implements the <xref:System.IDisposable> interface, making it convenient to define scopes with the `using` statement. 

By default, the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect encloses all instance public and internal methods with an implicit scope. That is, by default, public and internal methods define boundaries of logical operations. 

Unlike logical operations, scopes are generally nested. Scope nesting typically happens when a public method directly or indirectly invokes another public method. In general, only the outermost scope results in creating a logical operation. This is why, in general, logical operations form a flat structure.

Because they are visible to users, logical operations must be given a user-friendly name. PostSharp defines default names that are not user-friendly. The responsibility of generating operation names is implemented by the <xref:PostSharp.Patterns.Recording.OperationFormatter> class. You can provide your own <xref:PostSharp.Patterns.Recording.OperationFormatter> to generate operations names on demand, or you can set the name explicitly in source code for each operation. 

Scope names can be declaratively defined using the <xref:PostSharp.Patterns.Recording.RecordingScopeAttribute> custom attribute, or programmatically using the <xref:PostSharp.Patterns.Recording.Recorder.OpenScope(System.String,PostSharp.Patterns.Recording.RecordingScopeOption)> method. To learn more about operation names, see <xref:undoredo-operation-name>. 


## Atomic Operations

Atomic scopes are scopes whose changes are automatically rolled back when they do not complete successfully, typically when an exception occurs. The rollback is implemented using the undo mechanism. Atomic scopes are a concept similar to transactions, but multithreading is not taken into account. Therefore, other threads may see changes that have not been "committed", because the Recordable pattern does not have a notion of transaction isolation.

Atomic scopes cause composite operations to have a tree structure. However, the concept of atomic structure does not surface to the users. Therefore, from a user's perspective, the <xref:PostSharp.Patterns.Recording.Recorder.UndoOperations> and <xref:PostSharp.Patterns.Recording.Recorder.RedoOperations> collections still present linear lists of logical operations. 

Scopes are defined declaratively using the <xref:PostSharp.Patterns.Recording.RecordingScopeAttribute> custom attribute, or programmatically using the <xref:PostSharp.Patterns.Recording.Recorder.OpenScope(PostSharp.Patterns.Recording.RecordingScopeOption)> method. 


## Primitive Operations

The following table lists the primitive operations that are automatically appended to the <xref:PostSharp.Patterns.Recording.Recorder> object by the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect. 

| Class | Description |
|-------|-------------|
| <xref:PostSharp.Patterns.Recording.Operations.FieldOperation`1> | Represents the operation of setting a field to a different value. |
| <xref:PostSharp.Patterns.Recording.Operations.CollectionOperation`1> | Represents operations on collections. |
| <xref:PostSharp.Patterns.Recording.Operations.DictionaryOperation`2> | Represents operations on dictionaries. |
| <xref:PostSharp.Patterns.Recording.Operations.RecorderOperation> | Represents the operation of attaching or detaching an object to or from a <xref:PostSharp.Patterns.Recording.Recorder>.  |

In addition to these system-defined operations, it is possible to implement custom operations by deriving from the <xref:PostSharp.Patterns.Recording.Operation> abstract class. You can then use the <xref:PostSharp.Patterns.Recording.Recorder.AddOperation(PostSharp.Patterns.Recording.Operation)> method to append the custom operation to the <xref:PostSharp.Patterns.Recording.Recorder>. 

Logical operations, which are presented to the end user, are typically represented as instances of the <xref:PostSharp.Patterns.Recording.Operations.CompositeOperation> class. 


## Restore Points

Restore points act like bookmarks in the list of operations. They allow you to undo or redo operations up to a specific point. You can use the <xref:PostSharp.Patterns.Recording.Recorder.AddRestorePoint(System.String)> method to create a restore point. The method returns an instance of the <xref:PostSharp.Patterns.Recording.RestorePoint> class, which derives from the <xref:PostSharp.Patterns.Recording.Operation> class. Unlike other operations, you can safely remove a restore point from the history by using the <xref:PostSharp.Patterns.Recording.RestorePoint.Remove> method. 


## Implementing IEditableObject

You can use the <xref:PostSharp.Patterns.Recording.EditableObjectAttribute> custom attribute to automatically implement the <xref:System.ComponentModel.IEditableObject> interface. The implementation is based on the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect. It creates a <xref:PostSharp.Patterns.Recording.RestorePoint> when the `BeginEdit` method is invoked, removes the restore point upon `EndEdit`, and undoes changes up to the restore point when `CancelEdit` is called. 

Because of this implementation strategy, it is possible that `CancelEdit` actually cancels changes done to other objects that share the same <xref:PostSharp.Patterns.Recording.Recorder>. 


## Callback Methods

The <xref:PostSharp.Patterns.Recording.Recorder> will invoke the <xref:PostSharp.Patterns.Recording.IRecordableCallback.OnReplaying(PostSharp.Patterns.Recording.ReplayKind,PostSharp.Patterns.Recording.ReplayContext)> and <xref:PostSharp.Patterns.Recording.IRecordableCallback.OnReplayed(PostSharp.Patterns.Recording.ReplayKind,PostSharp.Patterns.Recording.ReplayContext)> methods of any recordable object implementing the <xref:PostSharp.Patterns.Recording.IRecordableCallback> interface, whenever the object is affected by an undo or redo operation. 

The order in which these methods are invoked on several objects is nondeterministic; in particular, the aggregation structure is not respected.

From callback methods, it is not allowed:

* to perform a change that would be recorded, for example, to set a field that has not been waived from recording with the <xref:PostSharp.Patterns.Recording.NotRecordedAttribute> custom attribute. 

* to invoke methods `Undo`, `Redo` or `AddRestorePoint` of the <xref:PostSharp.Patterns.Recording.Recorder> class. 


## Memory Consumption

The <xref:PostSharp.Patterns.Recording.Recorder.UndoOperations> and <xref:PostSharp.Patterns.Recording.Recorder.RedoOperations> collections hold strong references to all objects that have changes that can be undone or redone. This means that these objects cannot be garbage-collected and will remain in memory. 

You can define the maximal number of operations available for undo by using the <xref:PostSharp.Patterns.Recording.Recorder.MaximumOperationsCount> property. 


