This repository contains different projects which package usefule build targets to simplify certain tasks in your own projects.
Right now it contains two projects simplyfying the creation and packaging process of analyzers and source generators. Theses packages never contain any source code and only enhance your existing build

# Analyzer Support
Creating a good analyzer is hard enough. Delivering it to your end users is seemingly even harder. After countless hours of trial and error, I took inspiration in how Microsoft themselves ship their analyzers, by looking specifically at the repository for [System.Text.Json](https://github.com/dotnet/runtime/tree/main/src/libraries/System.Text.Json). I have extracted the resulting targets into these projects to help other developers struggling with the same issues.

## Packages
* [Cranks.Targets.GetAnalyzerPackFiles](https://www.nuget.org/packages/Cranks.Targets.GetAnalyzerPackFiles)
  Add this package to your analyzer projects.
  The package adds a new MS Build target to your project called `GetAnalyzerPackFiles`. When this target is called it returns a list of all the files which have to be packed into the analyzer folder of your libraries nuget package.
* [Cranks.Targets.GetAnalyzerPackFiles](https://www.nuget.org/packages/Cranks.Targets.AnalyzerReference)
  Add this package to a library project, in which you want to ship your analyzers or source generators with.
  This adds a new project reference type which you can use like this:
  `<AnalyzerReference Include="path\to\your\analyzer\project.csproj" Pack="true" ReferenceAnalyzer="false" />`
  It also includes new target called `IncludeAnalyzersInPackage` which gets called right before your project is packed. It then uses GetAnalyzerPackFiles in of all your `<AnalyzerReference>` projects and includes the relevant files into the analyzer directory of your nuget package.

## MS Build Variables
The added targets make use of certain variables. This are not required to set, but should be used anyway:
* `AnalyzerLanguage`
  defines the language of your analyzer, and hence in which folder it gets added in the nuget package, e.g. `cs`. Keep empty if your analyzer is language agnostic.
* `AnalyzerRoslynVersion`
  The Roslyn version the package targets. This defines the apis you can use in your project and depends on the minimum visual studio version you want to target
* `RoslynApiVersion`
  The api version you can use in your analyzer. The `<PackageReference>` of the `Microsoft.CodeAnalysis.CSharp.Workspaces` (or other languages you might want to be using) package should map to the value you enter here.

The `AnalyzerRoslynVersion` and `RoslynApiVersion` values depend on which version of visual studio you want to target and should always target the same version
To find which versions of VS correspond to which Roslyn version, take a look [.NET compiler platform package version reference](https://learn.microsoft.com/en-us/visualstudio/extensibility/roslyn-version-support).
`Major.Minor.Patch defines the RoslynApiVersion`, whereas `Major.Minor` restricts the AnalyzerRoslynVersion you want to target. 

As an example, when you want to create a source generator for C#, targeting at least Visual Studio 2022 RTM (17.0), add this to your project files:

```xml
<PropertyGroup>
    <AnalyzerLanguage>cs</AnalyzerLanguage>
    <AnalyzerRoslynVersion>4.0</AnalyzerRoslynVersion>
    <RoslynApiVersion>4.0.1</RoslynApiVersion>
</PropertyGroup>
```
