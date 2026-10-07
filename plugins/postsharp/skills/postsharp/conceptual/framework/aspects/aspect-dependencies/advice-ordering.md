---
uid: advice-ordering
redirect_from:
  - custompatterns/aspects/dependencies/advice-ordering
title: "Ordering Advices"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides guidance on ordering advices in PostSharp, explaining aspect dependencies, execution order, and the use of custom attributes for defining dependencies and ordering."
---
# Ordering Advices

The article <xref:aspect-dependencies> describes *aspect dependencies* and *aspect ordering*. Most of it also applies to advices. When we talk about the order of execution of aspects, we mean the order of execution of advices. Aspects themselves are never executed. 

Dependencies defined at aspect level implicitly apply to all advices. When developing a member-level aspect or a type-level aspect with advices (see <xref:member-level-aspects> and <xref:complex-aspects>), you can add dependencies directly to advice methods by annotating them with custom attributes of the namespace <xref:PostSharp.Aspects.Dependencies>. 

Note that all advices provided by an aspect are ordered in a single block. Suppose that a method is the target of advices `Aspect1.MethodA`, `Aspect1.MethodB` and `Aspect2.MethodC`. The next table shows valid and invalid orders: 

| Valid orders | Invalid orders |
|--------------|----------------|
| `Aspect1.MethodA`, `Aspect1.MethodB`, `Aspect2.MethodC`  | `Aspect1.MethodA`, `Aspect2.MethodC`, `Aspect1.MethodB`  |
| `Aspect1.MethodB`, `Aspect1.MethodA`, `Aspect2.MethodC`  | `Aspect1.MethodB`, `Aspect2.MethodC`, `Aspect1.MethodA`  |
| `Aspect2.MethodC`, `Aspect1.MethodA`, `Aspect1.MethodB`  |  |
| `Aspect2.MethodC`, `Aspect1.MethodB`, `Aspect1.MethodA`  |  |

## Ordering advices of the same aspect

Advices of the same aspect can be ordered using any custom attribute derived from <xref:PostSharp.Aspects.Dependencies.AspectDependencyAttribute>. 

Because advices of the same aspect instance are necessarily ordered as a block, it is appropriate to specify dependencies between aspect classes extensively, and specify ordering of advices only in the scope of the current aspect instance. The most appropriate dependency custom attribute for this purpose is <xref:PostSharp.Aspects.Dependencies.AdviceDependencyAttribute>, which accepts the name of the advice method as a parameter. 

## See Also

**Reference**

<xref:PostSharp.Aspects.Dependencies.AdviceDependencyAttribute>
