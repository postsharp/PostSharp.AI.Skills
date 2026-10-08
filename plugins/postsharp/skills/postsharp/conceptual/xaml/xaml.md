---
uid: xaml
title: "XAML"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a guide on implementing various aspects specific to XAML applications using PostSharp, including command pattern, dependency properties, and attached properties."
---
# XAML

This chapter covers the aspects that are specific to XAML applications.


## In this chapter

| Section | Description |
|---------|-------------|
| <xref:command> | This section shows how to implement the command pattern for the <xref:System.Windows.Input.ICommand> properties of your view model classes.  |
| <xref:dependency-property> | This section shows how to use the <xref:PostSharp.Patterns.Xaml.DependencyPropertyAttribute> aspect to turn simple properties into fully-featured dependency properties that support data binding, validation, styling, etc.  |
| <xref:attached-property> | This section shows how to use the <xref:PostSharp.Patterns.Xaml.AttachedPropertyAttribute> aspect to turn the simple properties into attached properties that are used in XAML to set the value on any object.  |

Additionally, if you are writing a XAML application, you may be interested in the following aspects:

| Chapter | Description |
|---------|-------------|
| <xref:inotifypropertychanged> | This chapter shows how to automatically implement the <xref:System.ComponentModel.INotifyPropertyChanged> interface without boilerplate code.  |
| <xref:contracts> | This chapter describes how to validate the value of your fields, properties and parameters with custom attributes. |
| <xref:weak-event> | This article shows how to turn normal events into weak events and prevent memory leaks. |


