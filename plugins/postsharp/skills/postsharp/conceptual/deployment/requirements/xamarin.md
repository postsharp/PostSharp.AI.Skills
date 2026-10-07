---
uid: xamarin
redirect_from:
  - deploymentconfiguration/deployment/xamarin
title: "Compatibility with Xamarin"
product: "postsharp"
categories: "PostSharp;AOP;Metaprogramming"
summary: "PostSharp 2024.1 and earlier support Xamarin through .NET Standard libraries, but not directly in a Xamarin application project. A custom linker configuration is needed to prevent removal of required code."
---
# Compatibility with Xamarin

PostSharp 2024.1 and earlier support Xamarin as a run-time platform only via .NET Standard. You can use PostSharp in your .NET Standard libraries and then reference these libraries in your Xamarin application project. Adding PostSharp directly to a Xamarin application project is not supported.

PostSharp 2025.0 and later do not support Xamarin.

For a complete list of PostSharp packages that support Xamarin, see <xref:requirements-20241>.


## Custom linker configuration

By default, all Xamarin applications use a linker in the Release build configuration. The purpose of the linker is to discard unused code and reduce the size of the application. Linking is based on static analysis, and it cannot correctly detect all the code used by PostSharp. To prevent the linker from removing the required code, you need a custom linker configuration in your project.

To add a custom linker configuration to your Xamarin application project, add the following XML file to the project and set the *Build Action* to *LinkDescription*. For more detailed information about the linker configuration see the Xamarin documentation: [Custom Linker Configuration](https://docs.microsoft.com/en-us/xamarin/cross-platform/deploy-test/linker). 

```xml
<linker>
  <assembly fullname="netstandard">
    <type fullname="*">
    </type>
  </assembly>
</linker>
```

## See Also

**Other Resources**

<xref:requirements>
<br>[Custom Linker Configuration](https://docs.microsoft.com/en-us/xamarin/cross-platform/deploy-test/linker)
<br>[Linking on Android](https://docs.microsoft.com/en-us/xamarin/android/deploy-test/linker)
<br>[Linking Xamarin.iOS Apps](https://docs.microsoft.com/en-us/xamarin/ios/deploy-test/linker)
