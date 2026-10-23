---
uid: constructor-dependencies
title: "Receiving Dependencies from the Constructors of the Target Class"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "Explains how an aspect requests a service with ImportDependencyAdviceInstance, so that the target class receives the service from its constructors, for instance from Microsoft.Extensions.DependencyInjection, and how RunTimeServiceLocator provides the service to objects created with new."
---
# Receiving Dependencies from the Constructors of the Target Class

Most applications create their services with a dependency injection container, such as Microsoft.Extensions.DependencyInjection, which passes the dependencies of each class to its constructor. An aspect can take part in this mechanism: it requests a service, and PostSharp adds a parameter of the service type to the constructors of the target class. The container then provides the service, and the advices of the aspect use it. The source code of the class does not mention the service.

This mechanism works with any aspect, including a statically scoped aspect such as a <xref:PostSharp.Aspects.MethodLevelAspect>, and with any container that calls the public constructor of the class. PostSharp Caching uses it to receive its caching service (see <xref:caching-dependency-injection>).


## Requesting a dependency

To make the target class receive a service from its constructors:

1. Implement the <xref:PostSharp.Aspects.Advices.IAdviceProvider> interface in the aspect (see <xref:advice-provider>).

2. In the <xref:PostSharp.Aspects.Advices.IAdviceProvider.ProvideAdvices(System.Object)> method, return an <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance>, and pass the type of the service to its constructor. The type must be a reference type, typically an interface.

3. In the advices, add a parameter of the service type, annotated with <xref:PostSharp.Aspects.Advices.ImportMemberAttribute>, without a member name (see [Importing members of the target type](xref:advice-parameter-binding#importing-members-of-the-target-type)). With the <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.Name> property of the request, you can also import the service by this name.

4. Register the service in the container, and let the container create the objects of the target class.

The *target class* is the class to which the aspect is applied, or the class that declares the member to which it is applied.


## Example

The following services write audit messages, count the calls and send notifications:

[!code-csharp[](../../../../code/samples/framework/aspects/dependency-injection/ConstructorDependency.cs#service)]

The `Audited` aspect requests three dependencies from the target class: a required `IAuditLog`, an optional `IMetrics`, and a lazy `INotifier`. After each successful execution of the target method, it writes an audit message, increments a counter when an `IMetrics` is available, and sends a notification:

[!code-csharp[](../../../../code/samples/framework/aspects/dependency-injection/ConstructorDependency.cs#aspect)]

The aspect is applied to a method of the `OrderService` class, whose constructor receives only its own dependency:

[!code-csharp[](../../../../code/samples/framework/aspects/dependency-injection/ConstructorDependency.cs#target)]

The build adds a parameter for each dependency to the public constructor of `OrderService`, so the container passes all the services. The lazy dependency is received as a `Lazy<INotifier>`, which the container must be able to create:

[!code-csharp[](../../../../code/samples/framework/aspects/dependency-injection/ConstructorDependency.cs#container)]

The container creates the `OrderService` without creating the notifier. The notifier is created when the advice first uses it, after the repository has saved the order. The output of the whole program is shown [below](#objects-created-without-a-container).


## Configuring the request

The <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance> class has the following properties:

| Property | Description |
|----------|-------------|
| <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.DependencyType> | The type of the service, passed to the constructor. |
| <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.IsRequired> | Whether the dependency is required. The default value is `true`. A required dependency is a mandatory constructor parameter: when the constructor receives `null` and no service is registered in <xref:PostSharp.Extensibility.RunTimeServiceLocator>, it throws an <xref:System.InvalidOperationException>. An optional dependency is an optional parameter whose default value is `null`, and its value can stay `null`. |
| <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.Name> | The name by which the advices import the dependency with `[ImportMember]`, whatever the names of the field and of the property, and the name of the property. Without a name, the advices import the dependency by its type. |
| <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.Visibility> | The visibility of the property that exposes the dependency, or `null` for the default value: <xref:PostSharp.Reflection.Visibility.Family?text=Family> (`protected`), or <xref:PostSharp.Reflection.Visibility.Private?text=Private> in a sealed class. A private dependency that is not lazy has no property. |
| <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.IsLazy> | Whether the service is resolved when it is first used instead of when the object is created. The default value is `false`. |

When several requests of one class are merged, the dependency is required when one of the requests is.

A lazy dependency is received as a <xref:System.Lazy`1> of the service type, stored in a field of this type, and exposed by a property of the service type that returns <xref:System.Lazy`1.Value>. The container must be able to create the <xref:System.Lazy`1>: Microsoft.Extensions.DependencyInjection, for instance, does not create it unless you register it.


## How the target class is transformed

For each target class, the request does the following at build time.

### The field

The request first looks for a member of the class that already holds the service, in the following order:

1. A field of exactly the service type, declared by the class, or by a base class when the class can access it, for instance a `protected` field.
2. A property that PostSharp created for the same dependency of a base class (see [The property](#the-property)).
3. A readable instance property of exactly the service type, declared by the class, or by a base class when the class can access its getter, for instance an auto-implemented property or a `protected` property of a base class. When several properties match, PostSharp reports the error <xref:la0286?text=LA0286>.

When the request finds such a member, the introduction is skipped, and the advices import this member. Nothing else changes: the class receives no new field, property or constructor parameter, and `new` remains valid. PostSharp trusts that the class or its base class initializes this member, as with a constructor parameter that the source code declares. The advices read the dependency through the member.

Otherwise, PostSharp introduces a private, read-only and non-serialized field of the service type.

### The property

When the dependency is not private, or when it is lazy (see [Configuring the request](#configuring-the-request)), the field is exposed by a getter-only property of the service type:

* Its visibility is <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.Visibility>, which is `protected` by default, or `private` in a sealed class.
* It is named after <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.Name>, or, without a name, after the service type without the `I` of an interface: `AuditLog` for `IAuditLog`. A number is appended when another member has this name.
* Derived classes, also in other assemblies, can read the dependency through this property.
* A derived class whose aspect requests the same dependency gets no field of its own: its advices import the property of the base class.

### The constructors

PostSharp transforms each instance constructor of the class, except the deserialization constructor:

* It appends one parameter for each dependency of the class and of its base classes, base classes first. When the constructor already has a parameter of exactly the service type, PostSharp uses this parameter and does not add one. This is how the source code can also receive the service.
* The constructor with the new parameters, called the *enhanced constructor*, gets the body, the custom attributes and the parameters of the original constructor, with their custom attributes and default values. Its visibility is the visibility of the original constructor.
* The original constructor becomes a *forwarder*, which calls the enhanced constructor and passes `null` for each dependency. A `public` forwarder becomes `protected internal`, and the other forwarders keep their visibility. A container therefore sees one public constructor, `new` still compiles in the project, and derived classes in other assemblies can still call the forwarder.

### The assignment

The enhanced constructor assigns the field just after the call to the base constructor, before the body of the constructor and before the aspects of the object are initialized, as C# assigns a field from a constructor parameter. When the argument is `null`, the constructor gets the service from <xref:PostSharp.Extensibility.RunTimeServiceLocator> (see [Objects created without a container](#objects-created-without-a-container)):

* For a required dependency, the constructor calls <xref:PostSharp.Extensibility.RunTimeServiceLocator.GetRequiredService*>. When no service of this type is registered, it throws an <xref:System.InvalidOperationException>, and the object is not created.
* For an optional dependency, the constructor calls <xref:PostSharp.Extensibility.RunTimeServiceLocator.GetService*>, and the dependency stays `null` when no service is registered.
* For a lazy dependency, the constructor creates a <xref:System.Lazy`1> that calls <xref:PostSharp.Extensibility.RunTimeServiceLocator.GetRequiredService*> when the dependency is first used. The exception is then thrown at that time, not by the constructor.

A constructor that calls `this(...)` passes its arguments to the other constructor.

### The derived classes

PostSharp also transforms the classes that derive from the target class, even when no aspect is applied to them: the classes of the same project, and the classes of other projects that PostSharp processes, for instance a project that derives from a class of a library. Their constructors receive the dependencies of their base classes, and their `base(...)` and `this(...)` calls pass them. A base constructor that declares the parameter of the service type in its source code is an exception: the derived class passes what its own source code chooses, and receives no parameter for it. A `base(...)` call that passes `null` to a parameter of the service type passes the dependency of the caller instead. When a derived class requires a dependency that its base class receives as optional, the derived class resolves it with <xref:PostSharp.Extensibility.RunTimeServiceLocator.GetRequiredService*> before it calls the base constructor. A record copies the dependency fields in its copy constructor.

### Equivalent code

The `OrderService` class of the example is transformed as if it were written as follows. The parameters are appended in the order of the requests. In C#, an optional parameter cannot precede a required one, but in the generated constructor it can:

<!-- not-compiled: equivalent of the code generated by PostSharp -->
```csharp
public class OrderService
{
    private readonly IOrderRepository repository;
    private readonly IAuditLog auditLog;
    private readonly IMetrics metrics;
    private readonly Lazy<INotifier> notifier;

    protected internal OrderService( IOrderRepository repository )
        : this( repository, null, null, null )
    {
    }

    public OrderService( IOrderRepository repository, IAuditLog auditLog, IMetrics metrics = null, Lazy<INotifier> lazy )
    {
        this.auditLog = auditLog ?? RunTimeServiceLocator.GetRequiredService<IAuditLog>();
        this.metrics = metrics ?? RunTimeServiceLocator.GetService<IMetrics>();
        this.notifier = lazy ?? new Lazy<INotifier>( RunTimeServiceLocator.GetRequiredService<INotifier> );
        this.repository = repository;
    }

    protected IAuditLog AuditLog => this.auditLog;

    protected IMetrics Metrics => this.metrics;

    protected INotifier Notifier => this.notifier.Value;

    public void PlaceOrder( int orderId )
    {
        this.repository.Save( orderId );
        AuditedAttribute.OnSuccess( /* the MethodBase of PlaceOrder */, this.AuditLog, this.Metrics, this.Notifier );
    }
}
```

In a static member, there is no object. An `[ImportMember]` parameter that imports the dependency therefore receives `null`.

Several requests of the same service type in one class, for instance from one aspect applied to several methods, give one field and one constructor parameter, when they have the same <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.Name>, <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.Visibility> and <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.IsLazy> values. These requests do not need to be ordered with aspect dependencies, even when their <xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance.IsRequired> values differ. Requests of different aspects that differ in another property must be ordered, otherwise PostSharp reports the warning <xref:ps0114?text=PS0114>.

> [!IMPORTANT]
> The request changes the public constructors of the target class and of its derived classes. When the class is public and used by other assemblies, this is a binary breaking change for them: recompile them. When PostSharp processes these assemblies, their derived classes receive the dependency automatically. Otherwise, their derived classes must call an enhanced constructor and pass the dependency themselves.


## Objects created without a container

When a constructor receives `null` for a dependency, which is the case when the source code creates the object with `new` and calls a forwarder, the constructor gets the service from <xref:PostSharp.Extensibility.RunTimeServiceLocator>. This class is a process-wide registry of service factories:

* <xref:PostSharp.Extensibility.RunTimeServiceLocator.RegisterService*> registers the factory of a service type, or replaces the factory registered for this type.
* <xref:PostSharp.Extensibility.RunTimeServiceLocator.TryRegisterService*> registers a factory only when no factory is registered for the type, without running the registered factory. A library uses it to provide a default service that the application can replace, before or after.
* <xref:PostSharp.Extensibility.RunTimeServiceLocator.GetRequiredService*> runs the factory, and throws an <xref:System.InvalidOperationException> when no factory is registered or when the factory returns `null`. The constructors call it for a required dependency.
* <xref:PostSharp.Extensibility.RunTimeServiceLocator.GetService*> runs the factory, and returns `null` when no factory is registered. The constructors call it for an optional dependency.
* <xref:PostSharp.Extensibility.RunTimeServiceLocator.ServiceProvider> is an <xref:System.IServiceProvider> over the same registry.

A factory that returns an object of another type causes an <xref:System.InvalidCastException>.

The factory runs each time a service is resolved, and its result is not cached. A factory can therefore return a singleton, a new object each time, or an object that depends on the context, for instance on an <xref:System.Threading.AsyncLocal`1> that the application sets.

The following code continues the example. The first object is created with `new` before any `IAuditLog` is registered, so its constructor throws. After the registration of `IAuditLog` and `INotifier`, `new` gets these services from the locator. `IMetrics` is not registered, so the optional dependency is `null`, and the advice increments no counter:

[!code-csharp[](../../../../code/samples/framework/aspects/dependency-injection/ConstructorDependency.cs#locator)]

The program prints:

[!code-text[](../../../../code/samples/framework/aspects/dependency-injection/ConstructorDependency.txt)]

<xref:PostSharp.Extensibility.RunTimeServiceLocator> is not a dependency injection container: it has no scopes, and it does not dispose the services. Register the services at startup, before the objects that need them are created. Each registration copies the registry, so registration is relatively slow, and resolution is fast and needs no lock.


## Initializing the object after its construction

An <xref:PostSharp.Aspects.Advices.OnInstanceConstructedAdvice> runs once the constructor of the most derived class has completed (see <xref:advice-parameter-binding>). At that time, the dependency is assigned, so the advice can import it with an `[ImportMember]` parameter, also when the aspect that requests the dependency is the aspect that has the advice. An object created with `new` gets the dependency from <xref:PostSharp.Extensibility.RunTimeServiceLocator> before the advice runs.


## Limitations

* The target class must be a class that has instance constructors. A struct, an interface or a delegate causes the error <xref:la0274?text=LA0274>.
* The service type must be a reference type that contains no generic parameters. Otherwise, PostSharp reports the error <xref:la0275?text=LA0275>.
* When an enhanced constructor would have the signature of another constructor of the class, PostSharp reports the error <xref:la0278?text=LA0278>. Remove one of the two constructors, usually the one that does not receive the service.
* When the class has several properties that could hold the service, PostSharp reports the error <xref:la0286?text=LA0286>. Remove all but one of these properties, or give the class a field of the service type that stores the dependency.
* Keyed services are not supported.
* A derived class in an assembly that PostSharp does not process is not transformed. It must call an enhanced constructor and pass the dependency itself.


## See Also

**Reference**

<xref:PostSharp.Aspects.Advices.ImportDependencyAdviceInstance>
<br><xref:PostSharp.Aspects.Advices.ImportMemberAttribute>
<br><xref:PostSharp.Aspects.Advices.IAdviceProvider>
<br><xref:PostSharp.Extensibility.RunTimeServiceLocator>

**Other Resources**

<xref:consuming-dependencies>
<br><xref:advice-provider>
<br><xref:advice-parameter-binding>
<br><xref:caching-dependency-injection>
<br>[Dependency injection in .NET](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
