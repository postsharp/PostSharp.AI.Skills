---
uid: advice-provider
title: "Providing Advices Dynamically"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how an aspect implements IAdviceProvider to import members of the target class, introduce interfaces and methods, request fields and request dependencies with code, when these advices depend on the target."
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
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance> | None | Requests a field in the target type: an existing field, or a new one. The advices of the aspect read and write it with `[ImportMember]` parameters. See [Requesting fields](#requesting-fields). |
| <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance> | None | Requests a service that the target class receives from its constructors, for instance from a dependency injection container. The advices of the aspect read it with `[ImportMember]` parameters. See <xref:constructor-dependencies>. |

Import advices and interface introductions require an aspect derived from <xref:PostSharp.Aspects.InstanceLevelAspect> (errors <xref:la0079?text=LA0079> and <xref:la0069?text=LA0069>). Field and dependency requests do not: any aspect can provide them, including a statically scoped aspect such as a <xref:PostSharp.Aspects.MethodLevelAspect>, because the advices receive the field through their parameters, not through a field of the aspect. The aspect fields and methods that these advices refer to are public members of the aspect class, as with the custom attributes, and the import fields are instance fields. An aspect field that receives imported members is set at run time, so mark it with <xref:PostSharp.Serialization.PNonSerializedAttribute> when the aspect is serialized with <xref:PostSharp.Serialization.PSerializableAttribute>.

Behaviors such as method boundary and interception advices cannot be provided this way. To apply them to members selected with code, use a <xref:PostSharp.Aspects.Advices.MethodPointcut> (see <xref:pointcuts>), or add aspects with <xref:PostSharp.Aspects.IAspectProvider> (see <xref:aspect-provider>).


## Example

The following aspect implements <xref:System.IDisposable> on its target class, and disposes all the fields of the class whose type implements <xref:System.IDisposable>. `ProvideAdvices` imports each of these fields into the `DisposableFields` list, and introduces the interface only when the class has such fields.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/DisposeFields.cs#aspect)]

When the aspect is applied to a class that has two fields of a disposable `Connection` type, disposing an instance of the class prints the following output:

[!code-text[](../../../../code/samples/framework/aspects/type-level-aspects/DisposeFields.txt)]


With <xref:PostSharp.Aspects.Advices.InterfaceOverrideAction.Ignore?text=InterfaceOverrideAction.Ignore>, a class that already implements <xref:System.IDisposable>, for instance through a base class, keeps its own `Dispose` method, and the aspect does not dispose its fields.


## Requesting fields

An aspect may need to store a value for each target object, for instance a flag, a counter or a cache. A statically scoped aspect has one instance for all the objects of a class, so it cannot keep this value itself. An <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance> requests a field in the target type instead: the type to which the aspect is applied, or the type that declares the member to which it is applied. The target type can be a class or a struct.

The request does not always introduce a field. A compatible field may already exist, for instance because the source code declares it or because another aspect introduced it. The request then says what to do: fail, skip the introduction, or introduce another field. The request does not initialize the field: a new field has the default value of its type.

The advices of the aspect instance that made the request receive the field through a parameter annotated with <xref:PostSharp.Aspects.Advices.ImportMemberAttribute>, by value, as `in`, `ref` or `out`, or as an <xref:PostSharp.Aspects.ILocationBinding`1> (see [Importing members of the target type](xref:advice-parameter-binding#importing-members-of-the-target-type)):

* When the request has a <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.Name>, the parameter imports the field by this name, whatever the real name of the field. The requested field wins over a member of the target type that has the same name.
* When the request has no name, the parameter imports the field by its type, without a name. The fields that the aspect instance requested are then the only candidates of this type.

The requested name is visible only to the aspect instance that made the request. Other aspects see an introduced field as a member of the type, by its real name or by its type.


### Example

The following aspect makes a method run only once for each object. It requests one new `bool` field for each method to which it is applied, without a name, and its entry advice imports this field by type with a `ref` parameter. The aspect is a <xref:PostSharp.Aspects.MethodLevelAspect>, so it is statically scoped, but the flag is stored in each object.

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/FieldRequest.cs#aspect)]

The aspect is applied to the `Open` and `Close` methods of a class that already has a `bool` field. <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.IntroduceAnother?text=IntroduceFieldCompatibleAction.IntroduceAnother> gives each method a field of its own, next to the `isOpen` field, which the aspect does not import:

[!code-csharp[](../../../../code/samples/framework/aspects/type-level-aspects/FieldRequest.cs#target)]

Each object opens and closes its connection only once:

[!code-text[](../../../../code/samples/framework/aspects/type-level-aspects/FieldRequest.txt)]


### Configuring the request

The <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance> class has the following properties:

| Property | Description |
|----------|-------------|
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.FieldType> | The type of the field, passed to the constructor. It can depend on the target, and it can be a closed generic type. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.Name> | The requested name, or `null`. A name finds an existing field of this name, and it is the name of a new field. Without a name, an existing field is found by its type, and a new field gets a generated name that cannot conflict with the source code. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> | What the request does when the target type already has a compatible field. See below. The default value is <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.Fail?text=Fail>. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenIncompatibleMemberExists> | What a request with a name does when a member has this name but is not a compatible field. See below. The default value is <xref:PostSharp.Aspects.Advices.IntroduceFieldIncompatibleAction.Fail?text=Fail>. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.Visibility> | The visibility of a new field. The default value is <xref:PostSharp.Reflection.Visibility.Private?text=Private>. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.IsStatic> | Whether the field is static. A static field of a generic type exists once for each closed generic type. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.IsReadOnly> | Whether a new field is read-only (`initonly`). The advices then import it by value or as `in` only. The default value is `false`. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.IsSerialized> | Whether a new field is serialized with the object. The default value is `true`. When it is `false`, the field is marked `NotSerialized`, which a serializer such as `BinaryFormatter` skips. |
| <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.CustomAttributes> | The custom attributes added to a new field, as <xref:PostSharp.Reflection.ObjectConstruction> objects. They are not added to an existing field. |

A new field has the <xref:System.Runtime.CompilerServices.CompilerGeneratedAttribute> custom attribute.

A field is *compatible* with the request when it has exactly the type of the request and the same kind (static or instance), and when it has the requested name, or any name when the request has no name. The candidates are the fields of the target type, the fields of its base types that the target type can access, and the fields introduced by earlier requests. Constants and the fields generated by the compiler are not candidates.

<xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> takes one of the following <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction> values:

* <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.Fail?text=Fail>, the default value: the build fails with the error <xref:la0271?text=LA0271>.
* <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.Ignore?text=Ignore>: the introduction is skipped, and the advices of the aspect import the compatible field. This is how several aspects share one field.
* <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.IntroduceAnother?text=IntroduceAnother>: a new field is introduced next to the compatible field. Its name is the requested name followed by a number, such as `handlers2`, or a generated name when the request has no name.

A member that has the requested name but is not a compatible field, such as a property, a constant or a field of another type, is an *incompatible member*. <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenIncompatibleMemberExists> takes one of the following <xref:PostSharp.Aspects.Advices.IntroduceFieldIncompatibleAction> values:

* <xref:PostSharp.Aspects.Advices.IntroduceFieldIncompatibleAction.Fail?text=Fail>, the default value: the build fails with the error <xref:la0272?text=LA0272>.
* <xref:PostSharp.Aspects.Advices.IntroduceFieldIncompatibleAction.IntroduceAnother?text=IntroduceAnother>: a new field is introduced, with the requested name followed by a number. The existing member is not renamed.

<xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenIncompatibleMemberExists> applies only to a request that has a name: a request without a name finds fields by their type, and its new field gets a generated name that cannot conflict with another member.

When no field is compatible and no member is incompatible, the request introduces a new field with the requested name, or with a generated name. A request without a name that finds several compatible fields causes the error <xref:la0273?text=LA0273>, unless <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> is <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.IntroduceAnother?text=IntroduceAnother>. Give it a name.

The number appended to a name depends on the order of the aspects. A field that other code must find by its name should therefore not set <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> or <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenIncompatibleMemberExists> to `IntroduceAnother`.


### Ordering several requests

PostSharp resolves the requests of a type after those of its base types, in the order of the aspects (see <xref:aspect-dependencies>). When two aspects request fields that can resolve to the same field (the same name, a numbered name, or no name with the same type and kind), the result depends on their order. Order these aspects with aspect dependencies, otherwise PostSharp reports the warning <xref:ps0114?text=PS0114>. A request declares the effect <xref:PostSharp.Aspects.Dependencies.StandardEffects.MemberIntroduction?text=StandardEffects.MemberIntroduction>, and the effect returned by <xref:PostSharp.Aspects.Dependencies.StandardEffects.GetMemberIntroductionEffect*> for its name, or for its type when it has no name.

The following requests do not need to be ordered:

* The requests of the same aspect instance, which are resolved in the order in which the aspect provides them.
* Equivalent requests, which have the same name, type, kind, visibility, serialization, read-only flag, custom attributes, <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> and <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenIncompatibleMemberExists>, when <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> is not <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.Fail?text=Fail>. With <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.Ignore?text=Ignore>, they share one field whatever their order. With <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.IntroduceAnother?text=IntroduceAnother>, only the numbers appended to the names depend on the order. This is the case of one aspect applied to several members of a class.


### Build messages

| Code | Severity | Meaning |
|------|----------|---------|
| <xref:la0268?text=LA0268> | Error | A field cannot be introduced into the target type, which is an interface, an enum or a delegate, or the aspect is applied to an assembly. |
| <xref:la0269?text=LA0269> | Error | The field type is not valid: a by-reference type, a pointer, `void`, or a type that contains generic parameters. A byref-like type, such as <xref:System.Span`1>, is valid only as an instance field of a `ref struct`. |
| <xref:la0270?text=LA0270> | Error | The requested name is empty or contains only white space. Set <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.Name> to `null` to request a field by its type. |
| <xref:la0271?text=LA0271> | Error | A compatible field exists, and <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> is <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.Fail?text=Fail>. |
| <xref:la0272?text=LA0272> | Error | An incompatible member has the requested name, and <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenIncompatibleMemberExists> is <xref:PostSharp.Aspects.Advices.IntroduceFieldIncompatibleAction.Fail?text=Fail>. |
| <xref:la0273?text=LA0273> | Error | The request has no name, several compatible fields exist, and <xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance.WhenCompatibleFieldExists> is not <xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction.IntroduceAnother?text=IntroduceAnother>. |
| <xref:la0279?text=LA0279> | Error | An instance field is requested in a type that has an explicit layout, where a new field would have no offset. Request a static field, or declare the field with its offset. |
| <xref:ps0114?text=PS0114> | Warning | Two aspects request fields that can resolve to the same field, and they are not ordered. |


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
<br><xref:PostSharp.Aspects.Advices.IntroduceFieldAdviceInstance>
<br><xref:PostSharp.Aspects.Advices.IntroduceFieldCompatibleAction>
<br><xref:PostSharp.Aspects.Advices.IntroduceFieldIncompatibleAction>
<br><xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance>

**Other Resources**

<xref:complex-aspects>
<br><xref:pointcuts>
<br><xref:code-injections>
<br><xref:members>
<br><xref:aspect-provider>
<br><xref:advice-parameter-binding>
<br><xref:constructor-dependencies>
