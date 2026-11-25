---
title: Organize projects for .NET Framework and .NET
description: Help for project owners who want to compile their solution against .NET Framework and .NET side by side.
author: conniey
ms.date: 07/02/2021
---
# Organize your project to support both .NET Framework and .NET

You can create a solution that compiles for both .NET Framework and .NET side by side. This article covers several project-organization options to help you achieve this goal. Here are some typical scenarios to consider when you're deciding how to set up your project layout with .NET. The list may not cover everything you want.

- [Combine existing projects and .NET projects into a single project](#replace-existing-projects-with-a-multi-targeted-net-project)

  **Benefits**:

  - Simplifies your build process by compiling a single project rather than multiple projects that each target a different .NET Framework version or platform.
  - Simplifies source file management for multi-targeted projects because you must manage a single project file. When adding or removing source files, the alternatives require you to manually sync these files with your other projects.
  - Easily generate a NuGet package for consumption.
  - Allows you to write code for a specific .NET Framework version by using compiler directives.

  **Drawback**:

  - Requires developers to use Visual Studio 2019 or a later version to open existing projects. To support older versions of Visual Studio, [keeping your project files in different folders](#support-vs) is a better option.

- <a name="support-vs"></a>[Keep all projects separate](#keep-existing-projects-and-create-a-net-project)

  **Benefits**:

  - Supports development on existing projects for developers and contributors who may not have Visual Studio 2019 or a later version.
  - Lowers the possibility of creating new bugs in existing projects because no code churn is required in those projects.

Consider [this example GitHub repository](https://github.com/dotnet/samples/tree/main/framework/libraries/migrate-library/). The figure below shows how this repository is laid out:

:::image type="content" source="media/project-structure/existing-project-structure.png" alt-text="Existing project structure diagram":::

The following sections describe several ways to add support for .NET based on the example repository.

## Replace existing projects with a multi-targeted .NET project

Reorganize the repository so that any existing *\*.csproj* files are removed and a single *\*.csproj* file is created that targets multiple frameworks. This is a great option, because a single project can compile for different frameworks. It also has the power to handle different compilation options and dependencies per targeted framework.

:::image type="content" source="media/project-structure/multi-targeted-project.png" alt-text="project that targets multiple frameworks diagram":::

For example code, see [GitHub](https://github.com/dotnet/samples/tree/main/framework/libraries/migrate-library-csproj/).

Changes to note are:

- Replacement of *packages.config* and *\*.csproj* with a new [.NET *\*.csproj*](https://github.com/dotnet/samples/tree/main/framework/libraries/migrate-library-csproj/src/Car/Car.csproj). NuGet packages are specified with `<PackageReference> ItemGroup`.

### Vary source code across target frameworks

When multi-targeting in a single project file with `<TargetFrameworks>`, your source code is compiled separately for each target framework. To handle APIs that differ between frameworks (such as APIs available in .NET Framework or modern .NET but not in .NET Standard), use preprocessor directives and SDK-defined conditional compilation symbols.

The .NET SDK automatically defines symbols matching each target framework (such as `NETFRAMEWORK`, `NETSTANDARD`, `NET`, `NET8_0`, `NET8_0_OR_GREATER`, and `NET48`):

# [C#](#tab/csharp)

```csharp
namespace MyLibrary
{
    public static class PlatformHelper
    {
        public static string GetPlatformInfo()
        {
#if NET8_0_OR_GREATER
            return $"Modern .NET: {System.Environment.Version}";
#elif NETFRAMEWORK
            return $"Running on .NET Framework: {System.AppDomain.CurrentDomain.FriendlyName}";
#else
            return "Running on .NET Standard fallback";
#endif
        }
    }
}
```

# [Visual Basic](#tab/vb)

```vb
Public Class PlatformHelper
    Public Shared Function GetPlatformInfo() As String
#If NET8_0_OR_GREATER Then
        Return $"Modern .NET: {System.Environment.Version}"
#ElseIf NETFRAMEWORK Then
        Return $"Running on .NET Framework: {System.AppDomain.CurrentDomain.FriendlyName}"
#Else
        Return "Running on .NET Standard fallback"
#End If
    End Function
End Class
```

---

You can also conditionally reference packages or project items per target framework in your project file using MSBuild conditions:

```xml
<ItemGroup Condition="'$(TargetFramework)' == 'net48'">
  <Reference Include="System.Web" />
</ItemGroup>

<ItemGroup Condition="'$(TargetFramework)' != 'net48'">
  <PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
</ItemGroup>
```

For more guidance and examples of multi-targeting, see [Cross-platform targeting - Multi-targeting](../../standard/library-guidance/cross-platform-targeting.md#multi-targeting) and [How to use preprocessor directives](../../csharp/language-reference/preprocessor-directives.md).

## Keep existing projects and create a .NET project

If there are existing projects that target older frameworks, you may want to leave these projects untouched and use a .NET project to target future frameworks.

:::image type="content" source="media/project-structure/separate-projects-same-source.png" alt-text=".NET project with existing projects in a different folder diagram":::

For example code, see [GitHub](https://github.com/dotnet/samples/tree/main/framework/libraries/migrate-library-csproj-keep-existing/).

The .NET and existing projects are kept in separate folders. Keeping projects in separate folders avoids forcing you to have Visual Studio 2019 or later versions. You can create a separate solution that only opens the old projects.

## See also

- [.NET porting documentation](index.md)
