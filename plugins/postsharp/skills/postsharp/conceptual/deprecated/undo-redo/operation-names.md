---
uid: undoredo-operation-name
redirect_from:
  - model/undoredo/undoredo-operation-name
title: "Customizing Undo/Redo Operation Names"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides detailed steps on how to customize Undo/Redo operation names in PostSharp, including default operation naming algorithm, setting operation name declaratively and dynamically, and using the OperationFormatter class."
---
# Customizing Undo/Redo Operation Names

> [!CAUTION]
> Undo/Redo was deprecated in PostSharp 2026.0 and will not be maintained. The API will be removed in the future.

The example of the previous sections displays the list of operations appearing in the two UI buttons. That list of operations references the setters on the different individual properties in a very technical manner, for instance the operation of setting the first name is named `set_FirstName`, according to the name of the property in source code. 

End users will want to see the operations described in meaningful business terms, not technical ones. This article will show you how to explicitly name the recording operations that will take place in your code.


## Default operation naming algorithm

From the end user's perspective, the undo/redo feature exposes a flat list of operations that can be undone or redone. From a system perspective, an operation is composed of changes to individual fields and collections. For instance, moving a picture on a design surface is seen as a single operation `Move` by the user, but it is composed of two changes in fields `x` and `y`. 

The following code example illustrates this:

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoOperationNames.cs#example)]

The `UndoHistory.Print` method prints the names of the operations of the <xref:PostSharp.Patterns.Recording.Recorder.UndoOperations> collection, from the oldest to the most recent:

[!code-text[](../../../code/samples/Model/UndoRedo/UndoRedoOperationNames.txt)]

By default, the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect will automatically open a new operation for any public or internal method unless the current <xref:PostSharp.Patterns.Recording.Recorder> already has an open operation. Therefore, invoking the `Move` method results in a single operation, even if it modifies two fields. Note that the `Move` method invokes the setters of public properties `X` and `Y`, which are themselves public methods, but they do not open new operations since they run from within the `Move` method. However, when properties `X` and `Y` are accessed from outside of the `Picture` class, new operations are created for the `set_X` and `set_Y` methods. 


## Setting the operation name declaratively

By default, the name of an operation is set to the name of the method. There are various ways to customize this name, and the easiest is to add a <xref:PostSharp.Patterns.Recording.RecordingScopeAttribute> custom attribute to the public method. 

In the following example, we are declaring a different name for the `Move` method: 

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoRecordingScope.cs#example)]

With that <xref:PostSharp.Patterns.Recording.RecordingScopeAttribute> added, the recorded operation will now have a name of `Moving the picture` instead of just `Move`. 


## Setting the operation scope and name dynamically

Setting the operation name declaratively is convenient but relatively rigid. When more flexibility is needed, you can use the <xref:PostSharp.Patterns.Recording.Recorder.OpenScope(System.String,PostSharp.Patterns.Recording.RecordingScopeOption)> method to control the creation and naming of scopes. 

In the following example, we will modify the `Move` method to include the target position in the operation description. 


### Naming an operation dynamically

1. Add the `[RecordingScope(RecordingScopeOption.Skip)]` custom attribute to the method, so that the method does not automatically define a new operation. 

    > [!NOTE]
    > This step is not required if you are starting the operation from a non-recordable object.
    > However, if you do not add this custom attribute to a method of a recordable object, the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect will automatically create a new scope to execute the method, and your call of the <xref:PostSharp.Patterns.Recording.Recorder.OpenScope(System.String,PostSharp.Patterns.Recording.RecordingScopeOption)> method will be ignored. 


2. Invoke the <xref:PostSharp.Patterns.Recording.Recorder.OpenScope(System.String,PostSharp.Patterns.Recording.RecordingScopeOption)> method and wrap the code you want to record in a `using` block. 



### Example

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoOpenScope.cs#example)]

After two calls of the `Move` method, the names of the undo operations are:

[!code-text[](../../../code/samples/Model/UndoRedo/UndoRedoOpenScope.txt)]


## Using the OperationFormatter class

Explicitly declaring the name for every operation would be a large and tedious task. It is possible to write your own naming engine and apply that set of naming rules across the entire application. To achieve this, derive your own implementation from the <xref:PostSharp.Patterns.Recording.OperationFormatter> class. 

In the following example, we will create a custom formatter that reads the operation name from the <xref:System.ComponentModel.DisplayNameAttribute> custom attribute and displays the value to which a property has been set. 


### Creating and registering a custom OperationFormatter

1. Create a new class and inherit from the <xref:PostSharp.Patterns.Recording.OperationFormatter> class. 


2. Create a constructor for the new formatter class.


3. Override the <xref:PostSharp.Patterns.Recording.OperationFormatter.FormatOperationDescriptor(PostSharp.Patterns.Recording.Operations.IOperationDescriptor)> method and write your custom logic for generating a custom operation name. 

    > [!NOTE]
    > Formatters create a chain of responsibility. If one formatter is unable to provide a name it will ask the next formatter in the chain to attempt to provide a name. To make the hand-off occur, the <xref:PostSharp.Patterns.Recording.OperationFormatter.FormatOperationDescriptor(PostSharp.Patterns.Recording.Operations.IOperationDescriptor)> method needs to return `null`. If it returns anything else, the chain is broken and the returned value is used as a name. 


4. Finally, you need to add your custom name formatter into the chain of responsibility.

    [!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoOperationFormatter.cs#register)]

    Because the <xref:PostSharp.Patterns.Recording.RecordingServices> is making use of a chain of responsibility, you are able to insert as many custom name formatters as you want. You are also able to determine their order of execution based on the order that you insert them into the chain of responsibility. 



### Example

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoOperationFormatter.cs#formatter)]

The <xref:PostSharp.Patterns.Recording.Operations.MethodExecutionOperationDescriptor.Method> property is `null` unless the <xref:PostSharp.Patterns.Recording.RecordableAttribute.ProvideMethodContext> property of the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect is `true`. Therefore, this formatter requires the following recordable class:

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoOperationFormatter.cs#customer)]

When a program sets the `FirstName` property, then calls the `Clear` method, the names of the undo operations are:

[!code-text[](../../../code/samples/Model/UndoRedo/UndoRedoOperationFormatter.txt)]

> [!NOTE]
> The `Method` and `Arguments` properties of the `MethodExecutionOperationDescriptor` class are `null` unless the <xref:PostSharp.Patterns.Recording.RecordableAttribute.ProvideMethodContext> property is set to `true`. Apply the aspect as `[Recordable(ProvideMethodContext = true)]` to the classes that this formatter must handle.


