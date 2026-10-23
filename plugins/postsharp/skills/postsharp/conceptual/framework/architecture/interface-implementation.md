---
uid: control-implementation
redirect_from:
  - custompatterns/constraints/tutorials/control-implementation
title: "Restricting Interface Implementation"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on how to use PostSharp's InternalImplementAttribute to restrict users from implementing an interface in their own classes, helping prevent issues when new members are added to the interface. "
---
# Restricting Interface Implementation

You may want to prevent users of an API from implementing an interface. They can consume the interface but not implement it in their own classes, so that you can later add new members to this interface without breaking the code of the users. If the interface must remain public, the programming language has no option to enforce this restriction. The <xref:PostSharp.Constraints.InternalImplementAttribute> attribute of PostSharp provides this restriction. 


## Adding the constraint to the interface

To restrict the implementation of a publicly declared interface, add [<xref:PostSharp.Constraints.InternalImplementAttribute>] to that interface. 

> [!NOTE]
> This procedure requires [Visual Studio Tools for Metalama and PostSharp](https://marketplace.visualstudio.com/items?itemName=PostSharpTechnologies.PostSharp) to be installed on your machine. You can, however, achieve the same results by editing the code and the project manually. 

1. Place the caret on the interface to which you want to add the attribute, and select "Add architectural constraint...".

    ![](InternalImplement_1.PNG)


2. Select "Prevent interface implementation in a different assembly" and select **Next**. 

    ![](InternalImplement_2.png)


3. Verify that you will be adding the <xref:PostSharp.Constraints.InternalImplementAttribute> attribute to the correct piece of code. 

    ![](InternalImplement_3.png)


4. When the download, installation and configuration of PostSharp have finished, close the wizard and look at the changes that were made to your codebase.

    ![](InternalImplement_4.png)


5. Notice that the only change in the code is the addition of the [<xref:PostSharp.Constraints.InternalImplementAttribute>] attribute. 

    [!code-csharp[](../../../code/samples/CustomPatterns/Constraints/InternalImplement/InternalImplement.Dependency.cs#interface)]


After that, implementing the interface that is decorated with <xref:PostSharp.Constraints.InternalImplementAttribute> from another assembly creates the build-time warning <xref:ar0101?text=AR0101>, unless the assembly of the interface grants access to that assembly with `InternalsVisibleToAttribute`. In the following code, `CustomerRepository` is in a different assembly than `ICustomerRepository`:

[!code-csharp[](../../../code/samples/CustomPatterns/Constraints/InternalImplement/InternalImplement.cs#implementation)]

The build reports the following warning:

[!code-text[](../../../code/samples/CustomPatterns/Constraints/InternalImplement/InternalImplement.build.txt)]

> [!NOTE]
> For this architectural validation to occur, PostSharp must process the project that implements the interface.


## Emitting an error instead of a warning

If a warning is not strict enough, you can change the output to a build-time error by setting the `Severity` property of <xref:PostSharp.Constraints.InternalImplementAttribute> to `Error`. 

[!code-csharp[](../../../code/samples/CustomPatterns/Constraints/InternalImplementError/InternalImplementError.Dependency.cs#interface)]

Now any implementation of the decorated interface in another assembly generates an error and fails the build of your project:

[!code-text[](../../../code/samples/CustomPatterns/Constraints/InternalImplementError/InternalImplementError.build.txt)]


## Ignoring warnings

If you implement a constrained interface in a separate assembly and you want to suppress the warning that is generated, apply the <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute. 

> [!NOTE]
> The <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute suppresses only warnings. If you have escalated the warnings to errors, those errors are still generated even if the <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute is present. 

To suppress the warning, add the <xref:PostSharp.Extensibility.SuppressWarningAttribute> attribute to the offending code. In this example, add the attribute to the class that implements the constrained interface. The warning for that specific implementation is then suppressed. All other locations that implement this interface still generate their warnings. 

> [!NOTE]
> You may wonder where the identifier <xref:ar0101?text=AR0101> comes from. <xref:PostSharp.Extensibility.SuppressWarningAttribute> works with any PostSharp warning, not only this one. Any build warning or error, whether from MSBuild, C# or PostSharp, has an identifier. To see identifiers in Visual Studio, open the **View** menu, click **Output**, and select **Show output from: Build**. The warnings are displayed with their identifiers. 

[!code-csharp[](../../../code/samples/CustomPatterns/Constraints/InternalImplement/InternalImplement.cs#suppress)]

In this example, the build reports the warning for `CustomerRepository` but not for `PreferredCustomerRepository`.

## See Also

**Reference**

<xref:PostSharp.Constraints.InternalImplementAttribute>
<br><xref:PostSharp.Extensibility.SuppressWarningAttribute>

**Other Resources**

<xref:control-visibility>
