---
uid: dependency-property
title: "Dependency Property"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document describes how to create and use XAML dependency properties in a PostSharp application. It provides examples of creating simple dependency properties, exposing the DependencyProperty object, validating values with contracts, and using a validation method."
---
# Dependency Property

XAML [dependency properties](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/dependency-properties-overview) extend the functionality of the CLR properties with features such as data binding, styling, and animation. Whenever you need to define a dependency property in your XAML application, you typically have to follow a strict implementation pattern and write a fair amount of boilerplate code. The <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> aspect allows you to create custom dependency properties much faster, without writing repetitive code. 


## Creating a simple dependency property


### Adding a dependency property to your class

1. Add a new property to your class with a chosen name, a type, and a public getter and setter.


2. Mark your new property with the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> attribute. 


After you have marked your property with the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> attribute, you can immediately start using advanced features in your XAML code. 


### Example

The following code snippet comes from a custom control named `ContactCard`. The snippet defines a dependency property named `Phone`. The custom control defines other dependency properties not listed here. 

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#phone)]

The dependency properties can be used in XAML for advanced features. In the following code snippet, we use data binding with the dependency properties of the `ContactCard` custom control. 

[!code-xml[](../../code/samples/XAML/DependencyPropertyExample.xaml)]


## Exposing the DependencyProperty object

If you want to manipulate the dependency property using the XAML API in C# or VB, you need to get the <xref:System.Windows.DependencyProperty> for this specific property. One solution is to use the <xref:PostSharp.Patterns.Xaml.DependencyPropertyServices.GetDependencyProperty(System.Type,System.String)> method, but this is long and error-prone. 

A better solution is to add a `public static` property of type <xref:System.Windows.DependencyProperty> with the same name as your dependency property plus the `Property` suffix. This property is picked up by the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> aspect and set to the proper <xref:System.Windows.DependencyProperty> value. 


### Example

The following example shows how to expose the <xref:System.Windows.DependencyProperty> for the `Phone` property. 

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#phone-registration)]

You can then use the `PhoneProperty` property, for instance to create a binding in C#:

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#set-binding)]


## Validating the value of the dependency property with contracts

PostSharp Code Contracts (see <xref:contracts>) provide a convenient way to validate the values of the dependency properties. To add the validation to your dependency property, you need to apply a contract attribute to that property. 


### Example

The following code snippet shows how to validate a dependency property using a code contract.

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#full-name)]


## Validating the value of the dependency property with a validation method

If you need more complex validation for your dependency property, you can implement it in a dedicated validation method. To define a validation method for the `Email` dependency property, add a new method named `ValidateEmail` to the same class where the property is declared. The method must accept one argument of a type that is assignable from the property type and return a `bool` value. 

The following list shows the method signatures you can use when implementing the validation method, where `TPropertyType` is the type of the dependency property and `TDeclaringType` is the class where your property is declared. 

* `static bool ValidatePropertyName(TPropertyType value)`
* `static bool ValidatePropertyName(DependencyProperty property, TPropertyType value)`
* `static bool ValidatePropertyName(TDeclaringType instance, TPropertyType value)`
* `static bool ValidatePropertyName(DependencyProperty property, TDeclaringType instance, TPropertyType value)`
* `bool ValidatePropertyName(TPropertyType value)`
* `bool ValidatePropertyName(DependencyProperty property, TPropertyType value)`

### Example

The following code snippet shows how to validate a dependency property using a validation method.

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#email)]


## Reacting to the changes of the dependency property value

The WPF property system can automatically notify you about the dependency property value changes via callback methods. This can be useful when, for example, you need to update the visual presentation of your custom UI control in response to a change of its property. This section shows how you can define a property change callback method with the dependency property pattern of PostSharp.

To define a property change callback method for the `PictureUrl` dependency property, add a new method named `OnPictureUrlChanged` to the same class where the property is declared. The method does not have to accept any arguments and must have a `void` return type. Implement your property change handling logic inside this new method. 

The following list shows the method signatures you can use when implementing the property change callback method. `TDeclaringType` is the class where your property is declared. 

* `static void OnPropertyNameChanged()`
* `static void OnPropertyNameChanged(DependencyProperty property)`
* `static void OnPropertyNameChanged(TDeclaringType instance)`
* `static void OnPropertyNameChanged(DependencyProperty property, TDeclaringType instance)`
* `void OnPropertyNameChanged()`
* `void OnPropertyNameChanged(DependencyProperty property)`

### Example

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#picture-url)]


## Implementing INotifyPropertyChanged

You may also want to notify the users of your class when a dependency property value changes. In this case, you would normally need to implement the <xref:System.ComponentModel.INotifyPropertyChanged> interface in your class and raise the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event. PostSharp helps you to automate this task using the <xref:inotifypropertychanged> pattern. To raise the <xref:System.ComponentModel.INotifyPropertyChanged.PropertyChanged> event every time any of the dependency properties in your class changes its value, mark your class with the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> attribute. 

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#contact-card)]

The sample program binds the properties of the control, then changes them. It prints the following output:

[!code-text[](../../code/samples/XAML/DependencyPropertyExample.txt)]


## Overriding the naming conventions

The <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> aspect follows a predefined naming convention when looking for methods and properties associated with the dependency property in your class. You can override the naming convention and choose your own member names by setting properties on the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute>. The following table shows the default naming convention and the properties used to override member names. 


### Dependency property pattern's naming conventions

| Member kind | Default name | Example | Override property |
|-------------|--------------|---------|-------------------|
| Value validation method | Validate *PropertyName*  | `ValidatePhoneNumber` | <xref:PostSharp.Patterns.Xaml.DependencyPropertyBaseAttribute.ValidateValueMethod> |
| Property changed callback method | On *PropertyName* Changed  | `OnPhoneNumberChanged` | <xref:PostSharp.Patterns.Xaml.DependencyPropertyBaseAttribute.PropertyChangedMethod> |
| Registration property | *PropertyName* Property  | `PhoneNumberProperty` | <xref:PostSharp.Patterns.Xaml.DependencyPropertyBaseAttribute.RegistrationProperty> |


### Example

[!code-csharp[](../../code/samples/XAML/DependencyPropertyExample.cs#notes)]

## See Also

**Reference**

<xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute>
<br><xref:System.Windows.DependencyProperty>
<br><xref:PostSharp.Patterns.Xaml.DependencyPropertyServices>
<br><xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>
<br><xref:System.ComponentModel.INotifyPropertyChanged>
<br><xref:PostSharp.Patterns.Contracts.NotEmptyAttribute>

**Other Resources**

[Dependency Properties Overview](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/dependency-properties-overview)
<br><xref:contracts>
<br><xref:inotifypropertychanged>
