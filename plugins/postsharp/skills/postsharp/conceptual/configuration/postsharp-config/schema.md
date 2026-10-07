---
uid: configuration-schema
redirect_from:
  - deploymentconfiguration/configuration/configuration-schema
title: "Configuration File Schema Reference"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "The document provides a detailed explanation of the schema for a PostSharp configuration file, including the elements it contains and the order in which they are processed."
---
# Configuration File Schema Reference

The basic format of a PostSharp configuration file is as follows:

```xml
<Project xmlns="http://schemas.postsharp.org/1.0/configuration" xmlns:x="http://schemas.postsharp.org/1.0/configuration">

  <!-- The following elements must appear in the proper order. -->  

  <License Value="[<license-key>|<url>](;[<license-key>|<url>])*"/>

  <Property Name="<string>" Value="<expression:string>" Overwrite="[true|false]" Sealed="[true|false]" Deferred="[true|false]" Condition="<expression:bool>" />
  
  <SearchPath Path="<expression:string>(;<expression:string>)*" ReferenceDirectory="<expression:string>" PathKind="[File|Directory]" Condition="<expression:bool>" />
  
  <Using File="<expression:string>" ProjectName="<expression:string>" Condition="<expression:bool>" />

  <SectionType LocalName="<string>" Namespace="<string>" />

  <Service TypeName="<string>" AssemblyFile="<string>" Condition="<expression:bool>" />

  <!-- The rest of the file contains extension elements defined using <SectionType /> elements. 
       PostSharp itself defines the following extension elements: -->

  <Multicast>
    <MyMulticastAspect MyAttributeName="<value>" xmlns="clr-namespace:<namespace>;assembly:<assembly>" x:Condition="<expression:bool>" />
  </Multicast>

  <Logging xmlns="clr-namespace:PostSharp.Patterns.Diagnostics;assembly:PostSharp.Patterns.Diagnostics" x:Condition="<expression:bool>">
    <Profiles>
      <LoggingProfile Name="<string>"
                      Description="<string>"
                      Role="<string>"
                      IncludeExecutionTime="<bool>"
                      ExecutionTimeThreshold="<int>"
                      IncludeAwaitedTask="<bool>"
                      IncludeSourceLineInfo="<bool>"
                      AllowDynamicEnabling="<bool>"
                      EnabledRecordKinds="<LogRecordKind>">
        <DefaultOptions>
          <LoggingOptions Level="<LogLevel>"
                          IncludeParameterType="<bool>"
                          IncludeGenericParameters="<bool>"
                          IncludeParameterName="<bool>"
                          IncludeParameterValue="<bool>"
                          IncludeReturnValue="<bool>"
                          IncludeThisValue="<bool>" />
        </DefaultOptions>
        <ExceptionOptions>
          <LoggingOptions Level="<LogLevel>"
                          IncludeParameterType="<bool>"
                          IncludeGenericParameters="<bool>"
                          IncludeParameterName="<bool>"
                          IncludeParameterValue="<bool>"
                          IncludeReturnValue="<bool>"
                          IncludeThisValue="<bool>" />
        </ExceptionOptions>
      </LoggingProfile>
    </Profiles>
  </Logging>

</Project>
```


## Schema elements

The configuration file includes these elements, described in detail in the following sections:

* [Project](#project)
* [License](#license)
* [SearchPath](#searchpath)
* [Using](#using)
* [SectionType](#sectiontype)
* [Property](#property)
* [Service](#service)
* [Multicast](#multicast)


## Project

This element is the root of the configuration file.


## License

This element allows loading one or more license keys.

| Attribute | Type | Description |
|-----------|------|-------------|
| Value | string | Required. A semicolon-separated list of license keys, or a URL to the license server. |


## SearchPath

This element adds a file or a directory to the list of paths in which PostSharp searches for assemblies and plug-ins.

| Attribute | Type | Description |
|-----------|------|-------------|
| Path | string expression | Required. A semicolon-separated list of files or directories that must be added to the path. |
| PathKind | `File` or `Directory`  | Optional. Specifies the kind of items contained in the `Path` property. If this property is not specified, the item kind is automatically determined for each individual item of the `Path` property.  |
| ReferenceDirectory | string expression | Optional. The directory from which relative paths in the `Path` property are resolved.  |
| Condition | boolean expression | Optional. `true` if the element is considered, `false` if it must be ignored.  |


## Using

This element imports another configuration file into the current project.

| Attribute | Type | Description |
|-----------|------|-------------|
| File | string expression | Required. Name of the file to be imported. Unless the name is qualified by a relative or absolute path, the file will be searched for using the search path. In this case, PostSharp searches for a file with the extension *psplugin* or *dll*.  |
| ProjectName | string | Optional. If a single *dll* includes several configurations, specifies which configuration should be loaded.  |
| Condition | boolean expression | Optional. `true` if the element is considered, `false` if it must be ignored.  |


## SectionType

This element defines custom sections for the current project. Custom sections can appear under the `Project` element under all system-defined elements. 

| Attribute | Type | Description |
|-----------|------|-------------|
| LocalName | string | Required. The local name of the XML element representing the custom section. |
| Namespace | string | Required. The namespace of the XML element representing the custom section. |


## Property

This element defines a property for the current project.

| Attribute | Type | Description |
|-----------|------|-------------|
| Name | string | Required. The property name. |
| Value | string expression | Required. The value that is assigned to the property. |
| Overwrite | boolean | Optional. `true` if the element will overwrite any previously-defined property of the same name, otherwise `false`. The default value is `true`.  |
| Sealed | boolean | Optional. `true` if an attempt to overwrite this property should result in an error, otherwise `false`. The default value is `false`.  |
| Deferred | boolean | Optional. `true` if the expression in the `Value` attribute should be dynamically evaluated every time the property value is requested, or `false` if the expression should be set at the time the property is defined. The default value is `false`.  |
| Condition | boolean expression | Optional. `true` if the element is considered, `false` if it must be ignored.  |


## Service

This element registers a service with the service locator for the current project.

| Attribute | Type | Description |
|-----------|------|-------------|
| TypeName | string | Required. The full name of the type that implements the service. This class must have a public parameterless constructor and implement the <xref:PostSharp.Extensibility.IService> interface.  |
| AssemblyFile | string expression | Optional. The path of the assembly defining the service class. If the attribute is not provided, the type will be searched for in the assembly that PostSharp is currently processing. |
| Condition | boolean expression | Optional. `true` if the element is considered, `false` if it must be ignored.  |


## Multicast

This element can be used to add aspects, policies or constraints to a project without adding them to the C# project as custom attributes. Adding elements to this section is equivalent to adding them to source code at assembly level.

The `Multicast` section is convenient to add aspects to several projects from a single file. 

For details regarding this section, see <xref:configuration-serialization>. 


## Order of processing of configuration files

Elements of configuration files are processed in the following order:

* [License](#license) elements are loaded. 

* [Property](#property) elements are loaded. Properties are evaluated at this moment unless they are marked for deferred evaluation. 

* [SearchPath](#searchpath) elements are loaded. 

* [Using](#using) elements are loaded and referenced plug-ins and configuration files are immediately loaded. 

* [SectionType](#sectiontype) elements are loaded. 

* [Service](#service) elements are loaded, but they are not yet instantiated. 

* Extension elements are loaded, but they are not evaluated at this moment.

* Finally, services and other tasks are instantiated and the project is executed.


