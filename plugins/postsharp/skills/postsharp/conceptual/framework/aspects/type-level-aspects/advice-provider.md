---
uid: advice-provider
title: "Providing Advices Dynamically"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how an instance-level aspect implements IAdviceProvider to import members of the target class and introduce interfaces and methods with code, when these advices depend on the target."
---
# Providing Advices Dynamically

An aspect usually declares its import and introduction advices with custom attributes, such as <xref:PostSharp.Aspects.Advices.ImportMemberAttribute>, <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute> and <xref:PostSharp.Aspects.Advices.IntroduceInterfaceAttribute> (see <xref:code-injections> and <xref:members>). A custom attribute is the same for every target. When the advices depend on the target, for instance when the aspect must import every field of a given type, the aspect implements the <xref:PostSharp.Aspects.Advices.IAdviceProvider> interface instead.

PostSharp calls the <xref:PostSharp.Aspects.Advices.IAdviceProvider.ProvideAdvices(System.Object)> method at build time, for each target of the aspect. The method receives the target as a reflection object, typically a <xref:System.Type>, and returns the advices to add to the aspect instance, as <xref:PostSharp.Aspects.Advices.AdviceInstance> objects.


## Advice instances

The following classes represent the advices that an aspect can provide:

| Class | Equivalent attribute | Advice |
|-------|----------------------|--------|
| <xref:PostSharp.Aspects.Advices.ImportLocationAdviceInstance> | <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> on a field of type <xref:PostSharp.Aspects.Advices.Property`1> | Imports a field or property of the target class into an aspect field of type <xref:PostSharp.Aspects.ILocationBinding>, or of a collection type such as `List<ILocationBinding>`, with one advice per member. A property can also be imported into a <xref:PostSharp.Aspects.Advices.Property`1> field, as with the attribute, but a field cannot. |
| <xref:PostSharp.Aspects.Advices.ImportMethodAdviceInstance> | <xref:PostSharp.Aspects.Advices.ImportMemberAttribute> on a field of a delegate type | Imports a method of the target class into a delegate field of the aspect. |
| <xref:PostSharp.Aspects.Advices.IntroduceMethodAdviceInstance> | <xref:PostSharp.Aspects.Advices.IntroduceMemberAttribute> | Introduces a public method of the aspect class into the target class. |
| <xref:PostSharp.Aspects.Advices.IntroduceInterfaceAdviceInstance> | <xref:PostSharp.Aspects.Advices.IntroduceInterfaceAttribute> | Introduces an interface, which the aspect class implements, into the target class. |

Import advices and interface introductions require an aspect derived from <xref:PostSharp.Aspects.InstanceLevelAspect> (errors LA0079 and LA0069). The aspect fields and methods that these advices refer to are public members of the aspect class, as with the custom attributes, and the import fields are instance fields. An aspect field that receives imported members is set at run time, so mark it with <xref:PostSharp.Serialization.PNonSerializedAttribute> when the aspect is serialized with <xref:PostSharp.Serialization.PSerializableAttribute>.

Behaviors such as method boundary and interception advices cannot be provided this way. To apply them to members selected with code, use a <xref:PostSharp.Aspects.Advices.MethodPointcut> (see <xref:pointcuts>), or add aspects with <xref:PostSharp.Aspects.IAspectProvider> (see <xref:aspect-provider>).


## Example

The following aspect implements <xref:System.IDisposable> on its target class, and disposes all the fields of the class whose type implements <xref:System.IDisposable>. `ProvideAdvices` imports each of these fields into the `DisposableFields` list, and introduces the interface only when the class has such fields.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/DisposeFields.cs#aspect)]

When the aspect is applied to a class that has two fields of a disposable `Connection` type, disposing an instance of the class prints the following output:

[!code-text[](../../../../code/samples/framework/aspects/type-level-aspects/DisposeFields.txt)]


With <xref:PostSharp.Aspects.Advices.InterfaceOverrideAction.Ignore?text=InterfaceOverrideAction.Ignore>, a class that already implements <xref:System.IDisposable>, for instance through a base class, keeps its own `Dispose` method, and the aspect does not dispose its fields.


## Extension blocks

`ProvideAdvices` receives reflection objects, which represent the code that the compiler generates. For C# 14 extension blocks, this code includes metadata types and members that do not exist in the source code. Do not select them: use <xref:PostSharp.Reflection.ReflectionHelper.IsExtensionBlockMetadata*> to recognize them (see <xref:extension-blocks-multicast>).


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.IAdviceProvider>
<br><xref:PostSharp.Aspects.Advices.AdviceInstance>
<br><xref:PostSharp.Aspects.Advices.ImportLocationAdviceInstance>
<br><xref:PostSharp.Aspects.Advices.ImportMethodAdviceInstance>
<br><xref:PostSharp.Aspects.Advices.IntroduceMethodAdviceInstance>
<br><xref:PostSharp.Aspects.Advices.IntroduceInterfaceAdviceInstance>

**Other Resources**

<xref:complex-aspects>
<br><xref:pointcuts>
<br><xref:code-injections>
<br><xref:members>
<br><xref:aspect-provider>
