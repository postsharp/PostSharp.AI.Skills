---
uid: undoredo-recorder
redirect_from:
  - model/undoredo/undoredo-recorder
title: "Assigning Recorders Manually"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides instructions on how to manually assign recorders in PostSharp by either overriding the default RecorderProvider or attaching a recorder manually to objects where needed."
---
# Assigning Recorders Manually

> [!CAUTION]
> Undo/Redo was deprecated in PostSharp 2026.0 and will not be maintained. The API will be removed in the future.

By default, all recordable objects are attached to the global <xref:PostSharp.Patterns.Recording.Recorder> exposed on the <xref:PostSharp.Patterns.Recording.RecordingServices.DefaultRecorder> property. There is nothing you have to do to make this happen. There may be circumstances where you want to create and assign your own recorder to the undo/redo process. There are two different ways that you can accomplish this. 


## Overriding the default RecorderProvider

By default, the <xref:PostSharp.Patterns.Recording.RecordableAttribute> aspect attaches an object to a <xref:PostSharp.Patterns.Recording.Recorder> as soon as its constructor exits. To determine which <xref:PostSharp.Patterns.Recording.Recorder> should be used, the aspect uses the <xref:PostSharp.Patterns.Recording.RecordingServices.RecorderProvider> service. By default, this service always serves the global instance that is also exposed on the <xref:PostSharp.Patterns.Recording.RecordingServices.DefaultRecorder> property. 

You can override this automatic assignment to inject your own <xref:PostSharp.Patterns.Recording.RecorderProvider> into the process. 


### Using a custom RecorderProvider

1. Create a class derived from the <xref:PostSharp.Patterns.Recording.RecorderProvider> class. 


2. Implement the chaining constructor. The <xref:PostSharp.Patterns.Recording.RecorderProvider> that you inherited from requires a <xref:PostSharp.Patterns.Recording.RecorderProvider> as a constructor parameter. This constructor parameter facilitates the chain of responsibility for providers that can be run when a <xref:PostSharp.Patterns.Recording.Recorder> is requested. To keep the chain of responsibility intact your custom <xref:PostSharp.Patterns.Recording.RecorderProvider> will need to accept a <xref:PostSharp.Patterns.Recording.RecorderProvider> in its constructor and pass that to the base constructor. 


3. Override the <xref:PostSharp.Patterns.Recording.RecorderProvider.GetRecorderCore(System.Object)> method. 


4. Insert an instance of your custom <xref:PostSharp.Patterns.Recording.RecorderProvider> class into the chain of responsibility by assigning it to the <xref:PostSharp.Patterns.Recording.RecordingServices.RecorderProvider>. 

    [!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoRecorderProvider.cs#register)]


> [!NOTE]
> <xref:PostSharp.Patterns.Recording.RecordingServices.RecorderProvider> is a chain of responsibility. As such, if a <xref:PostSharp.Patterns.Recording.RecorderProvider.GetRecorderCore(System.Object)> method returns `null` then the chain will move on to the next <xref:PostSharp.Patterns.Recording.RecorderProvider> and attempt to get a <xref:PostSharp.Patterns.Recording.Recorder> to use. 

By overriding the default <xref:PostSharp.Patterns.Recording.RecorderProvider> you are able to assign a custom <xref:PostSharp.Patterns.Recording.Recorder> across the entire application. 


### Example

[!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoRecorderProvider.cs#provider)]


## Attaching a recorder manually

The second way that you can add a <xref:PostSharp.Patterns.Recording.Recorder> to objects is to manually assign them when, and where, they are needed. 


### Assigning a Recorder to an object manually

1. Set the <xref:PostSharp.Patterns.Recording.RecordableAttribute.AutoRecord> property to `false` for that class. 

    [!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoManualRecorder.cs#invoice)]

    > [!NOTE]
    > By disabling <xref:PostSharp.Patterns.Recording.RecordableAttribute.AutoRecord> you are telling the <xref:PostSharp.Patterns.Recording.RecordingServices> that this object should not be included in recordings unless the recording is explicitly declared in your code. 


2. Create a new instance of a <xref:PostSharp.Patterns.Recording.Recorder> and attach the object to it using the <xref:PostSharp.Patterns.Recording.Recorder.Attach(System.Object)> method. 

    [!code-csharp[](../../../code/samples/Model/UndoRedo/UndoRedoManualRecorder.cs#attach)]

    You can then use the <xref:PostSharp.Patterns.Recording.Recorder.Detach(System.Object)> method to remove the <xref:PostSharp.Patterns.Recording.Recorder> from the object in question. 


> [!NOTE]
> An object must always have the same <xref:PostSharp.Patterns.Recording.Recorder> as its parent has unless the parent has no <xref:PostSharp.Patterns.Recording.Recorder> assigned. Because of this, whenever a <xref:PostSharp.Patterns.Recording.Recorder> is assigned to an object, all of the child objects will have that same <xref:PostSharp.Patterns.Recording.Recorder> assigned to them. However, if you detach a child object from its parent the child object's assigned <xref:PostSharp.Patterns.Recording.Recorder> will not be detached. For more information about parent-child relationships, see <xref:aggregatable>. 


