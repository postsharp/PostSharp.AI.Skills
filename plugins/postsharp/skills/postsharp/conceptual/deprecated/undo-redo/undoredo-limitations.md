---
uid: undoredo-limitations
redirect_from:
  - model/undoredo/undoredo-limitations
title: "Limitations of Undo/Redo"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document discusses the limitations of the PostSharp product, including its intrinsic single-threaded nature and lack of support for async methods in recordable classes."
---

# Limitations of Undo/Redo

## Essentially Single-Threaded

> [!CAUTION]
> Undo/Redo was deprecated in PostSharp 2026.0 and will not be maintained. The API will be removed in the future.

The <xref:PostSharp.Patterns.Recording.Recorder> class is intrinsically single-threaded. You can use recordable objects in a multithreaded context, but you should make sure that objects that share the same recorder are not accessed concurrently from several threads. Note that this is a limitation of the undo/redo concept, not a limitation of our implementation. 


## No Support for Async Methods

Async methods are not supported as units of undoable operations. In short, it means that you cannot have async methods in recordable classes unless you make the method non-recorded with `[RecordingScope(RecordingScopeOption.Skip)]` (see <xref:PostSharp.Patterns.Recording.RecordingScopeAttribute>). Otherwise, the build fails with error <xref:dom026?text=DOM026>. 


