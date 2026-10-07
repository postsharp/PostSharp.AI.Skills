---
uid: aggregatable-adding-programmatically
redirect_from:
  - model/aggregatable/aggregatable-adding-programmatically
title: "Rule-Based Annotation"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document explains how to use rule-based annotation in PostSharp to programmatically annotate fields as a child or a reference, particularly useful for auto-generated fields or large numbers of fields."
---
# Rule-Based Annotation

There are times where you cannot or do not want to add custom attributes manually.

For instance, you cannot add any custom attribute to an auto-generated field backing a XAML element. The WPF designer generates C# code from your XAML files and stores them in *i.g.cs* files hidden in the *obj* folder. You cannot modify these files directly. 

Another scenario is when you have a large amount of fields and do not want to annotate each of them individually.

Field rules allow you to annotate a field as a child or a reference programmatically, without adding a custom attribute to each field manually.

In the following example, let us consider a class where PostSharp shows the error message *COM002: “Field/property InvoiceLine.product must be annotated with a custom attribute [Child], [Reference] or [Parent].”*:

```csharp
[Aggregatable]
public class InvoiceLine
{
  private Product product;

  public decimal Amount { get; set; }
}
```


## Marking all fields as references by default

1. Create a class derived from the <xref:PostSharp.Patterns.Model.TypeAdapters.FieldRule> class. 

    ```csharp
    public class ReferenceFieldRule : FieldRule
    ```


2. Override the <xref:PostSharp.Patterns.Model.TypeAdapters.FieldRule.GetRelationshipInfo(System.Reflection.FieldInfo)> method. PostSharp calls the <xref:PostSharp.Patterns.Model.TypeAdapters.FieldRule.GetRelationshipInfo(System.Reflection.FieldInfo)> method for each field that is not annotated with the <xref:PostSharp.Patterns.Model.ChildAttribute>, <xref:PostSharp.Patterns.Model.ReferenceAttribute> or <xref:PostSharp.Patterns.Model.ParentAttribute> custom attribute. The <xref:PostSharp.Patterns.Model.TypeAdapters.FieldRule.GetRelationshipInfo(System.Reflection.FieldInfo)> method allows you to specify the field relationship by returning a <xref:PostSharp.Patterns.Model.RelationshipInfo> instance. 

    ```csharp
    public override RelationshipInfo? GetRelationshipInfo(FieldInfo field)
    {
        return new RelationshipInfo(RelationshipKind.None, RelationshipKind.Reference);
    }
    ```


3. Decorate your project’s assembly with a <xref:PostSharp.Patterns.Model.TypeAdapters.RegisterFieldRuleAttribute> custom attribute to activate your field rule. 

    ```csharp
    [assembly: RegisterFieldRule(typeof(ReferenceFieldRule))]
    ```

    > [!CAUTION]
    > You have to mark each project assembly to make your FieldRule active in the whole solution.


    > [!NOTE]
    > PostSharp has two built-in rules: one rule for auto-generated WinForms fields and one rule for auto-generated XAML fields.

    ```xml
    <Window x:Class="WpfApp.MainWindow"
            xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
            xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
        <Grid>
            <Button x:Name="myButton" />
        </Grid>
    </Window>
    ```

In this example, the WPF designer generates a `myButton` field in a `MainWindow` class. Both the `MainWindow` class and the `Button` type of the `myButton` field derive from <xref:System.Windows.UIElement>. The built-in rule annotates the `myButton` field as a reference automatically. 

## See Also

**Reference**

<xref:PostSharp.Patterns.Model.TypeAdapters.FieldRule>
<br><xref:PostSharp.Patterns.Model.TypeAdapters.RegisterFieldRuleAttribute>
