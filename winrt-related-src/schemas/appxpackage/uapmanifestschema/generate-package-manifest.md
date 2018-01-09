---
author: laurenhughes
title: How Visual Studio generates an app package manifest
description: When you build an app, Visual Studio generates an associated package manifest. Learn how this is generated from your app's information. 
ms.author: lahugh
ms.date: 11/29/2017
ms.topic: article
ms.prod: windows
ms.technology: winrt-reference
keywords: windows 10, uwp, schema, package manifest, Visual Studio
---

# How Visual Studio generates an app package manifest
When you build a project with Visual Studio, Visual Studio generates a package manifest (AppxManifest.xml), which contains information the system needs to deploy, display, or update a Universal Windows Platform (UWP) app.

There are two varieties of app package manifest files that you'll encounter if developing an app with Visual Studio

- **Package.appxmanifest**  
  This is an XML style file that developers use to configure the details of an app, such as publisher information, logos, processor architectures, etc. This is an easily configurable, temporary version of the app package manifest used during app development.
- **AppxManifest.xml**  
  This file is generated by the Visual Studio build process and is based on the information in the Package.appxmanifest file. This is the final version of the app package manifest used with published and sideloaded apps. If any updates are made to the Package.appxmanifest file, you must rebuild the project to see the updates in the AppxManifest.xml file.

For an overview of the packaging process, see [Package a UWP app with Visual Studio](https://docs.microsoft.com/windows/uwp/packaging/packaging-uwp-apps).

## Validating the app manifest 

Before you can publish your app, you must fix any errors that cause any of the Visual Studio validation checks to fail. When Visual Studio generates the manifest, Visual Studio validates your app in the following ways:
- **Syntactic validation**  
  Visual Studio confirms whether all data in the app manifest conforms to the app manifest schema.   
- **Semantic validation**  
  Visual Studio provides guidance on expected data, based on the context of the information.   

> [!NOTE] 
> If these sections don’t mention the field that you’re looking for, it’s generated from either data that may have been configured separately or a default value from the manifest schema.

## Generating the manifest content 
Visual Studio populates the fields in the following tables when it generates the AppxManifest.xml file for the app package.

### Identity 
The [`Identity`](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/element-identity) section of the app manifest contains the following fields.

| Field | Description |
|-------|-------------|
| Name | The name of the package, which is populated differently in the following scenarios: <ul><li>By default, the value of this field is a generated GUID.</li><li>If you associate the app with the Microsoft Store, or if you invoke the **Store -> Create App Packages...** command and then sign in with a developer account, the value of this field is retrieved from the associated app in the Microsoft Store or Dev Center.</li><li>If you invoke the **Store -> Create App Packages...** command but do not sign in with a developer account, the value of this field is taken from the source manifest. </li></ul> |
| Publisher | The name of the publisher. This name gets populated differently in the following scenarios: <ul><li>By default, the value of this field is your user name.</li><li>If you associate the app with the Microsoft Store, or if you invoke the **Store -> Create App Packages...** command and then sign in with a developer account, the value of this field is the publisher associated with the account.</li><li>If you invoke the **Store -> Create App Packages...** command but do not sign in with a developer account, the value of this field matches the subject field of the test certificate you used to sign the app package.</li></ul> Visual Studio only supports the common name (CN) form for the publisher and will add the prefix "CN=" to publisher field in the manifest. |
| Version | The version of the app being built. This is typically incremented each time the app is has been modified and packaged. To ensure that the `Version` is incremented correctly, use the dialog provided when you invoke **Store -> Create App Packages...** to make updates. |
| ProcessorArchitecture | A value that's generated based on the build configuration that you specified for the project. If project references or file references in the project target a different specific architecture than the app package, a build error is thrown, and you must change the target architecture of the app package to work for all references. |


Here's an example of the `Identity` output XML:

```xml
<Identity Name="Microsoft.UWPAppExample"
          Publisher="CN=Microsoft Corporation"
          Version="1.0.0.0"
          ProcessorArchitecture="x86" />
```

### Properties 
The [`Properties`](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/element-properties) section of the app manifest contains the fields in the following table.


| Field | Description |
|-------|-------------|
| PublisherDisplayName | This string is populated differently in the following scenarios: <ul><li>By default, the value of this field is your user name.</li><li>If you associate the app with the Microsoft Store, or if you invoke the **Store -> Create App Packages...** command and then sign in with a developer account, tha value of this field matches the PublisherDisplayName string associated with your developer account.</li><li>If you invoke the **Store -> Create App Packages...** command but do not sign in with a developer account, the value of this field is your user name, unless you specify otherwise in the Package.appxmanifest file.</li></ul> |
| DisplayName | This string gets populated differently in the following scenarios: <ul><li>By default, the value of this field is the name of the project.</li><li>If you associate the app with the Microsoft Store, or if you invoke the **Store -> Create App Packages...** command and then sign in with a developer account, the value of this field is populated according to the following rules: <ul><li>If you specify this value in the source manifest and the value starts with `@` (which indicates that you want to localize this value), the value of this field will match what you specified.</li><li>If the selected app has only one name, the value will be that name. </li><li>If the selected app has multiple names but the source manifest isn’t localized, the value is set to the display name in the source manifest. Otherwise, the value is set to the first reserved name.</li></ul></li> <li>If you invoke the **Store -> Create App Packages...** command but do not sign in with a developer account, the value of this field is taken from the source manifest.</li></ul>|
| Logo | A Visual Studio template will use `Assets\StoreLogo.png` by default. This value should be customized by the developer in the Package.appxmanifest file. |

Here's an example of the `Properties` output XML:

```xml
<Properties>
    <DisplayName>UWP App Example</DisplayName>
    <PublisherDisplayName>Microsoft Corporation</PublisherDisplayName>
    <Logo>Assets\StoreLogo.png</Logo>
</Properties>
```


### Application 
A app manifest can contain multiple [`Application`](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/element-application) elements, each of which has a display name that appears on the tile in the client. The `Application` section of the app manifest contains the fields in the following table.

| Field | Description |
|-------|-------------|
| Id | This string gets populated differently in the following scenarios:<ul><li>By default, the value of this field is the name of the project. </li><li>If you associate the app with the Microsoft Store, or if you invoke the **Store -> Create App Packages...** command and then sign in with a developer account, the value of this field is the app name for the selected app if the `/Properties[@DisplayName]` and the `/Applications/Application[@DisplayName]` in the source manifest match. Otherwise, the value remains the same as in the source manifest.</li><li>If you invoke the **Store -> Create App Packages...** command but do not sign in with a developer account, the value of this field is the same as in the source manifest.</li></ul> |
| Executable | The value of this field is the output name of the project assembly. The executable token $targetnametoken$.exe that's used in the source manifest file (Package.appxmanifest) is replaced with the actual file name when the manifest is built. |
| EntryPoint | This value is based on the generated `Executable` and `Id` values. |

Example `Application` output:

```xml
<Applications>
    <Application Id="App" Executable="UWPAppExample.exe" EntryPoint="UWPAppExample.App">
        <!-- Other elements configured within the Application, such as Extensions, VisualElements, etc. -->
</Applications>
```

### PackageDependency 
The [`PackageDependency`](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/element-packagedependency) section contains all the Windows component library dependencies for this package. For example, if your project has a reference to WinJS, Visual Studio retrieves the package identity information of the dependencies when the manifest is generated. Visual Studio then populates this section with the `Name` and `MinVersion` fields for each dependent package. 


### Windows Runtime registration extensions 
You can implement Windows Runtime components for your apps, but you'll need to register those components with the operating system for them to run correctly. To register a Windows Runtime component, you must put the registration information in the WinMD files and in the app manifest. If a project implements a Windows Runtime component, the build output of the project will contain a WinMD file. Visual Studio extracts the Windows Runtime registration information from the WinMD file and generates the appropriate [`Extension`](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/element-extension) elements in the app manifest.

The system supports two forms of servers: .dll servers (in-process) and .exe servers (out-of-process). These servers require similar but different registration information that must be copied into the app manifest. Visual Studio supports generating manifest only for .dll servers, and the DLLServer extension is required to register .dll servers. The following values in the app manifest are taken from the WinMD files to construct the DLLServer Extension:

- DllPath 
- ActivatableClassId 
- ThreadingModel 
- ActivatableClass (ActivatableClassId attribute) 

Here's an example of the output XML: 

```xml
<extension category="Microsoft.Windows.ActivatableClass">
    <dllServer>
        <dllPath>Fabrikam.dll</dllPath>
        <activatableClass activatableClassId="Fabrikam.MyClass" threadingModel="sta" />
    </dllServer>
</extension>
```

For more information on this topic, see [Windows Runtime components](https://docs.microsoft.com/windows/uwp/winrt-components/). 

### Resources 
The [`Resources`](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/element-resources) section contains an entry for each language that the application supports. You must have at least one resource language specified in the app manifest. Visual Studio automatically generates the list of supported languages based on the localization information in the project. The resource language token "x-generate" that's used in the source manifest file (Package.appxmanifest) is replaced with the actual language code when the manifest is built. Here's an example of the output XML:

```xml
<Resources>
    <Resource Language="en-us">
    <Resource Language="fr-fr">
</Resources>
```

The first entry in the list is the default language for the app.

### TargetDeviceFamily 
The [`TargetDeviceFamily`](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/element-targetdevicefamily) section contains the following fields:
- Name
- MinVersion   
- MaxVersionTested   

These elements are populated from MSBuild properties.


## See also
[Package a UWP app with Visual Studio](https://docs.microsoft.com/windows/uwp/packaging/packaging-uwp-apps)  
[App package architectures](https://docs.microsoft.com/windows/uwp/packaging/device-architecture)  
[Windows 10 package manifest schema reference](https://docs.microsoft.com/uwp/schemas/appxpackage/uapmanifestschema/schema-root)  