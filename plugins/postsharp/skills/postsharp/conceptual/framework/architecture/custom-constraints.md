---
uid: custom-constraints
redirect_from:
  - custompatterns/constraints/tutorials/custom-constraints
title: "Developing Custom Architectural Constraints"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to develop custom architectural constraints in PostSharp, including scalar and referential constraints. It provides step-by-step instructions on creating these constraints and ensuring they are consistently implemented across a codebase."
---
# Developing Custom Architectural Constraints

Applications often adopt custom design patterns that must be respected across all modules. Custom design patterns have the same benefits as standard ones, but they are specific to your application. For instance, the team could decide that every class derived from `BusinessRule` must have a nested class named `Factory`, derived from `BusinessRulesFactory`, with a public default constructor. 

Even line-by-line code reviews can miss violations of the pattern. PostSharp lets you create custom architectural constraints to detect them. The constraints that you write can verify anything that you can query using reflection.

There are two kinds of constraints: *scalar constraints* and *referential constraints*. 

A scalar constraint applied to a declaration is checked when that declaration is compiled, so in only one assembly. Scalar constraints typically validate a type, member or parameter.

A referential constraint applied to a declaration can be checked many times: once for the assembly where that declaration is, and once for each assembly that uses (references) that declaration. Referential constraints typically validate the *usage* of a type, member or parameter. 


## Creating a scalar constraint

Start with a scalar constraint that verifies the first condition of the `BusinessRule` design pattern: any class derived from `BusinessRule` must have a nested class named `Factory`. You can model this condition as a scalar constraint that applies to any class derived from `BusinessRule`. Therefore, you create a type-level scalar constraint, apply it to the `BusinessRule` class, and use attribute inheritance to apply the constraint automatically to all derived classes. 

1. Create a class that inherits from the <xref:PostSharp.Constraints.ScalarConstraint> class in PostSharp. 

    ```csharp
    using System; 
    public class BusinessRulePatternValidation : ScalarConstraint 
    { 
    }
    ```


2. Specify the kind of code element that this validation aspect targets by adding the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> attribute. In this case, the validation must occur on types only, and inheritance must be enabled. <xref:PostSharp.Extensibility.MulticastInheritance.Strict> means that `BusinessRulePatternValidation` is applied not only to the class that it annotates but also to all of its subclasses, even in other assemblies. 

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRulePatternValidation : ScalarConstraint 
    { 
    }
    ```


3. Override the <xref:PostSharp.Constraints.ScalarConstraint.ValidateCode(System.Object)> method. 

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRulePatternValidation : ScalarConstraint 
    { 
        public override void ValidateCode(object target) 
        { 
        } 
    }
    ```


4. Create a rule that checks that there is a nested type named `Factory`. The `target` parameter of the <xref:PostSharp.Constraints.ScalarConstraint.ValidateCode(System.Object)> method is of type `object`. The type of the value passed in this parameter depends on the target kind that you declare in the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> attribute. For `MulticastTargets.Class`, the value is a `Type`. To use the target for validation, first cast it to that type. 

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRulePatternValidation : ScalarConstraint 
    { 
        public override void ValidateCode(object target) 
        { 
            var targetType = (Type) target; 
     
            if ( targetType.GetNestedType("Factory") == null ) 
            { 
               // warning 
            } 
        } 
    }
    ```

    > [!NOTE]
    > Valid types for the `target` parameter of the <xref:PostSharp.Constraints.ScalarConstraint.ValidateCode(System.Object)> method include `Assembly`, `Type`, `MethodInfo`, `ConstructorInfo`, `PropertyInfo`, `EventInfo`, `FieldInfo` and `ParameterInfo`. 


5. Write a warning about the broken rule to the Output window in Visual Studio.

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRulePatternValidation : ScalarConstraint 
    { 
        public override void ValidateCode(object target) 
        { 
            var targetType = (Type)target; 
     
            if (targetType.GetNestedType("Factory") == null) 
            { 
                Message.Write( 
                    targetType, SeverityType.Warning, 
                    "2001", 
                    "The {0} type does not have a nested type named 'Factory'.", 
                    targetType); 
            } 
        } 
    }
    ```


6. Attach the rule to the code that needs to be protected. In this example, add the rule to the `BusinessRule` class. 

    ```csharp
    [BusinessRulePatternValidation] 
    public class BusinessRule 
    { 
        // No nested Factory class: the constraint emits a warning for this class and for any subclass without one.
    }
    ```

    > [!NOTE]
    > This example applies the constraint to only one class. To apply a constraint to large portions of your codebase, see <xref:attribute-multicasting>. 


7. Build the project. A warning appears in the Output window of Visual Studio.

    ![](CustomArchConstraint_warning.PNG)

    > [!NOTE]
    > Due to a bug, the error code may not be shown in the Error list.


8. If a warning is not strict enough, you can change the rule so that it emits a build-time error instead. To do this, change the <xref:PostSharp.Extensibility.SeverityType> in `Message.Write` to `Error`. 

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRulePatternValidation : ScalarConstraint 
    { 
        public override void ValidateCode(object target) 
        { 
            var targetType = (Type)target; 
     
            if (targetType.GetNestedType("Factory") == null) 
            { 
                Message.Write( 
                targetType, SeverityType.Error, 
                "2001", 
                "The {0} type does not have a nested type named 'Factory'.", 
                targetType); 
            } 
        } 
    }
    ```

    ![](CustomArchConstraint_error.PNG)


With this technique, you can create rules or restrictions based on many different criteria and implement validation for several design patterns.

Projects must adhere to the principles of the project team. Manual verification fails at some point. As in other areas of the development process, you should automate the verification and enforcement of these principles. Custom architectural constraints provide the flexibility and verification that you need to achieve this goal.


## Creating a referential constraint

Now create a referential constraint that verifies the second condition of the `BusinessRule` design pattern: the `BusinessRule` class can be used only in the `Controllers` namespace. You can model this condition as a referential constraint. 

1. Create a class that inherits from the <xref:PostSharp.Constraints.ReferentialConstraint> class in PostSharp. 

    ```csharp
    public class BusinessRuleUseValidation : ReferentialConstraint 
    { 
    }
    ```


2. Declare that this aspect works only on types by adding the <xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute> attribute to the class. You will apply this class to `BusinessRule` to prevent all code in the project from using `BusinessRule` or its subclasses, unless the code is in the `Controllers` namespace. 

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRuleUseValidation : ReferentialConstraint 
    { 
    }
    ```


3. Override the <xref:PostSharp.Constraints.ReferentialConstraint.ValidateCode(System.Object,System.Reflection.Assembly)> method. In this case, the target is the `BusinessRule` class or one of its subclasses. The assembly is the assembly that is currently searched for violations. 

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRuleUseValidation : ReferentialConstraint 
    { 
        public override void ValidateCode(object target, Assembly assembly) 
        { 
        } 
    }
    ```


4. Create the rule that checks for the use of the default constructor of the `BusinessRule` type in all code. 

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRuleUseValidation : ReferentialConstraint 
    { 
        public override void ValidateCode(object target, Assembly assembly) 
        { 
            var targetType = (Type) target; 
            var usages = ReflectionSearch 
                    .GetMethodsUsingDeclaration(targetType.GetConstructor(new Type[0]));
     
            foreach (MethodUsageCodeReference methodUsageCodeReference in usages)
            {
                if (!methodUsageCodeReference.UsingMethod.DeclaringType.Namespace.Contains("Controllers"))
                {
                    // warning
                }
            }
        }
    }
    ```

    > [!NOTE]
    > The rule uses the <xref:PostSharp.Reflection.ReflectionSearch> helper class that PostSharp provides. This class, along with others, extends the built-in reflection functionality of .NET and can be used outside of aspects as well. The method <xref:PostSharp.Reflection.ReflectionSearch.GetMethodsUsingDeclaration(System.Reflection.MemberInfo)> already searches the current assembly, so the `assembly` parameter of `ValidateCode` is not needed. 


5. Write a warning message to the Output window of Visual Studio.

    ```csharp
    [MulticastAttributeUsage(MulticastTargets.Class, Inheritance = MulticastInheritance.Strict)] 
    public class BusinessRuleUseValidation : ReferentialConstraint 
    { 
        public override void ValidateCode(object target, Assembly assembly) 
        { 
            var targetType = (Type) target; 
            var usages = ReflectionSearch 
                    .GetMethodsUsingDeclaration(targetType.GetConstructor(new Type[0]));
     
            foreach (MethodUsageCodeReference methodUsageCodeReference in usages)
            {
                if (!methodUsageCodeReference.UsingMethod.DeclaringType.Namespace.Contains("Controllers"))
                {
                    Message.Write( 
                        targetType, SeverityType.Warning,  
                        "2002", 
                        "The {0} type contains a reference to '{1}' which can only be referenced from Controllers.",
                        methodUsageCodeReference.UsingMethod.DeclaringType,
                        target);
                }
            }
        }
    }
    ```


6. Attach the referential constraint to the `BusinessRule` class. 

    ```csharp
    [BusinessRuleUseValidation] 
    public class BusinessRule 
    { 
        // Now, only code in the Controllers namespace can call this class's constructor.
    }
    ```


7. Test the rule by adding nonconforming code to your project.

    ```csharp
    namespace PostSharp.Architecture.Repositories 
    { 
        public class AccountRepository 
        { 
            public void AddAccount(string name) 
            { 
                var businessRule = new BusinessRule(); 
            } 
        } 
    }
    ```


8. Build the project. A warning appears in the Output window in Visual Studio.

    ![](ReferentialConstraint_Warning.PNG)

    > [!NOTE]
    > If a warning is not strict enough, you can change the <xref:PostSharp.Extensibility.SeverityType> to `Error`. When the rule is broken, an error then appears in the Output window of Visual Studio and the build fails. 


> [!CAUTION]
> PostSharp constraints operate at the lowest level. For instance, checking the relationships of a type with the rest of the code does not implicitly check the relationships of the methods of this type. Also, checking the relationships of namespaces is not possible.
>
> You can use custom attribute multicasting to apply a constraint to a large number of types, for instance all types of a namespace. But this results in one constraint instance for every type, method and field in this namespace. Although this has no impact on run time, it can severely affect build time. For this reason, the current version of PostSharp Constraints is not suitable to check isolation (layering) of namespaces at large scale.

Referential constraints let you declare architectural design patterns in your code. These patterns are then documented in the codebase, which makes them easy for the development team to find, and they are continually verified.


## Validating the constraint itself

You now have scalar and referential constraints that consistently enforce certain architectural rules in your codebase. One thing is missing.

So far, you can attach your architectural constraints to any code element in your projects. This may not be appropriate. For example, the scalar constraint `BusinessRulePatternValidation` may be valid only on classes in the `Models` namespace. 

To ensure that this constraint is enforced only on classes in the `Models` namespace:

1. Open the `BusinessRulePatternValidation` class that you created earlier. 


2. Override the <xref:PostSharp.Constraints.Constraint.ValidateConstraint(System.Object)> method. 


3. Write the validation logic to ensure that this constraint is only applied to classes in the `Models` namespace. 

    > [!NOTE]
    > When the <xref:PostSharp.Constraints.Constraint.ValidateConstraint(System.Object)> method returns `true`, PostSharp applies the constraint to the target code element. When it returns `false`, PostSharp does not apply the constraint to the target code element. 


Now, when the `BusinessRulePatternValidation` attribute is applied to a class that is not in the `Models` namespace of your project, no warning or error is added to the Visual Studio Output window. 

When the attribute is applied to a class in the `Models` namespace and that class does not pass the rules of the constraint, the warning or error that indicates this architectural failure is still displayed. 

## See Also

**Reference**

<xref:PostSharp.Extensibility.MulticastAttributeUsageAttribute>
<br><xref:PostSharp.Constraints.ScalarConstraint>
<br><xref:PostSharp.Extensibility.SuppressWarningAttribute>
<br><xref:PostSharp.Extensibility.SuppressWarningAttribute.MessageId>
<br><xref:PostSharp.Extensibility.SuppressWarningAttribute.Reason>

**Other Resources**

<xref:attribute-multicasting>
<br><xref:multicast-inheritance>
