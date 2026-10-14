---
uid: command
title: "Command"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed guide on using PostSharp's CommandAttribute aspect to implement commands in MVVM pattern-based GUI applications, reducing boilerplate code and simplifying the implementation of the `CanExecute` logic."
---
# Command

In GUI applications built using the MVVM pattern, it is common for the view model class to define commands that the user can perform in the associated XAML view. These commands are implemented as nested classes of the view model classes, and require a lot of boilerplate code.

The <xref:PostSharp.Patterns.Xaml.CommandAttribute> aspect allows you to implement commands in your view model classes without adding nested classes, which greatly reduces boilerplate code. The aspect also integrates with the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect, which makes it much easier to implement the `CanExecute` logic. 


## Creating a simple command


### Adding a command named LoadContactsList to your view model class

1. Implement the command logic in the `ExecuteLoadContactsList` method in your view model class. This method is the equivalent of the <xref:System.Windows.Input.ICommand.Execute(System.Object)> method. 

    Optionally, the `Execute` method can have one parameter of any type, which becomes the command parameter. 


2. Add a `LoadContactsListCommand` property to your view model class and mark it with the <xref:PostSharp.Patterns.Xaml.CommandAttribute> attribute. The property must be of type <xref:System.Windows.Input.ICommand> and have both a getter and a setter. 



### Example

The following code snippet, from a view model class annotated with <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute>, defines a command named `LoadContactsList`. 

[!code-csharp[](../../code/samples/XAML/Command.cs#load)]

After adding a new command to your view model class, you can bind it to a UI control in your XAML code.

[!code-xml[](../../code/samples/XAML/Command.xaml)]

The sample program binds this view to the view model and executes the commands. It prints the following output:

[!code-text[](../../code/samples/XAML/Command.txt)]


## Determining whether your command can be executed

The UI controls in the GUI application can be enabled or disabled based on what actions are available to the user in the current state of the application. This feature is usually implemented by the <xref:System.Windows.Input.ICommand.CanExecute(System.Object)> method. 

The <xref:PostSharp.Patterns.Xaml.CommandAttribute> aspect provides a clean way to determine the availability of the actions. For each command in your view model class, you can implement a `CanExecute` property or method that validates whether this command can be currently executed. 


### Adding a CanExecute property

To allow the view to determine the availability of the `CreateContact` command, add a `public bool` property named `CanExecuteCreateContact` to the same class. 

[!code-csharp[](../../code/samples/XAML/Command.cs#create)]

The `CanExecute` properties associated with the commands support the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect. Therefore, if you add the <xref:PostSharp.Patterns.Model.NotifyPropertyChangedAttribute> aspect to the view model class, the UI will be notified of changes in the `CanExecute` properties. 


### Adding a CanExecute method

A limitation of `CanExecute` properties is that they cannot depend on the input argument of the command. For example, with the `DeleteContact` command, you may want to validate whether the current user has the permission to delete the currently selected contact. If your availability logic depends on the input argument, you must use a `CanExecute` method instead of a property. 

To allow the UI to determine the availability of the `DeleteContact` command, add a `CanExecuteDeleteContact` method to the view model class. The method must have one parameter and return a `bool` value. The parameter type must be assignable from the type of the command's input argument. 

[!code-csharp[](../../code/samples/XAML/Command.cs#delete)]


## Overriding naming conventions

The <xref:PostSharp.Patterns.Xaml.CommandAttribute> aspect follows a predefined naming convention when looking for methods and properties associated with the command in your view model class. You can override the naming convention and choose your own member names by setting properties on the <xref:PostSharp.Patterns.Xaml.CommandAttribute>. The following table shows the default naming convention and the properties used to override member names. 


### Command pattern's naming conventions

| Member kind | Default name | Example | Override property |
|-------------|--------------|---------|-------------------|
| Command property | *CommandName* -or- *CommandName* Command  | `UpdateContact` -or- `UpdateContactCommand`  | N/A |
| Execute method | Execute *CommandName* -or- *CommandName*  | `ExecuteUpdateContact` -or- `UpdateContact` | <xref:PostSharp.Patterns.Xaml.CommandAttribute.ExecuteMethod> |
| CanExecute method | CanExecute *CommandName* -or- Can *CommandName*  | `CanExecuteUpdateContact` -or- `CanUpdateContact` | <xref:PostSharp.Patterns.Xaml.CommandAttribute.CanExecuteMethod> |
| CanExecute property | CanExecute *CommandName* -or- Can *CommandName*  | `CanExecuteUpdateContact` -or- `CanUpdateContact` | <xref:PostSharp.Patterns.Xaml.CommandAttribute.CanExecuteProperty> |

The following example shows a command that does not follow the naming conventions.

[!code-csharp[](../../code/samples/XAML/Command.cs#update)]

## See Also

**Reference**

<xref:PostSharp.Patterns.Xaml.CommandAttribute>
<br><xref:System.Windows.Input.ICommand>

**Other Resources**

<xref:inotifypropertychanged>
