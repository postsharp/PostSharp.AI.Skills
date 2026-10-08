---
uid: threading-model-compatibility
redirect_from:
  - threading/threading-model-compatibility
title: "Compatibility of Threading Models"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document presents a compatibility matrix showing which threading models can be used as children based on the parent model in PostSharp. Also, it explains how assignment between objects with threading models works."
---
# Compatibility of Threading Models

This table shows which threading models you can use as children based on the model of the parent.


## Compatibility Matrix

| Parent↓ Child→ | Actor | Freezable | Immutable | Private | Reader-Writer Synchronized | Synchronized | Thread Affine | Thread Unsafe |
|----------------|-------|-----------|-----------|---------|----------------------------|--------------|---------------|---------------|
| Actor | No | Yes (Own) | Yes (Own) | Yes (Shared) | No | No | No | Yes (Shared) |
| Freezable | No | Yes (Own) | Yes (Own) | No | No | No | No | No |
| Immutable | No | Yes (Own) | Yes (Own) | No | No | No | No | No |
| Reader-Writer Synchronized | No | Yes (Own) | Yes (Own) | Yes (Shared) | Yes (Shared) | No | No | No |
| Synchronized | No | Yes (Own) | Yes (Own) | Yes (Shared) | Yes (Shared) | Yes (Shared) | No | Yes (Shared) |
| Thread Affine | No | Yes (Own) | Yes (Own) | Yes (Shared) | Yes (Shared) | Yes (Shared) | No | Yes (Shared) |
| Thread Unsafe | Yes (Own) | Yes (Own) | Yes (Own) | Yes (Shared) | Yes (Shared) | Yes (Shared) | No | Yes (Shared) |

*Own* means that the child keeps its own concurrency controller. *Shared* means that the child uses the concurrency controller of its parent.

A Private object (see <xref:PostSharp.Patterns.Threading.PrivateThreadAwareAttribute>) uses the concurrency controller of its parent. Its children must be compatible with the threading model of that parent.

When you assign an object with a threading model to another object with a threading model as a child, the assignment statement will block until the thread has full access to both objects. For a Synchronized object, that means it has the lock. For a ReaderWriterSynchronized object, that means it has the write lock. The same is true for an assignment which removes the parent-child link from two objects.

## See Also

**Other Resources**

<xref:aggregatable>
