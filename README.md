# CentralPackageVersions
A demonstration of how to centrally manage NuGet package versions.

## How projects reference packages
Individual projects are only allowed to specify which packages they reference and cannot specify a version.

```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" />
  <PackageReference Include="Microsoft.Build.Framework" />
</ItemGroup>
```

There is an MSBuild target that runs before build which detects if a project is specifying a package version and logs an error to fail the build.  The error contains information about which file the version should be specified in.
```
Build FAILED.

"D:\CentralPackageVersions\CentralPackageVersions.sln" (restore target) (1) ->
(ApplyPackageVersions target) ->
  D:\CentralPackageVersions\src\ClassLibrary\ClassLibrary.csproj  : error : The package reference
  'Newtonsoft.Json' should not specify a version.  Please specify the version in
  'D:\CentralPackageVersions\PackageVersions.props'.
```
## Central package definition
In this example, a central MSBuild property file named [PackageVersion.props](PackageVersions.props) defines package versions.  The list includes explicity references as well as the [new implicit references](https://aka.ms/sdkimplicitrefs).
See nuget [version-ranges](https://docs.microsoft.com/en-us/nuget/create-packages/dependency-versions#version-ranges) for supported versioning formats. 
```xml
<ItemGroup>
  <PackageVersion Include="Microsoft.Build.Framework" Version="[14.3.0]" />
  <PackageVersion Include="Microsoft.NET.Test.Sdk" Version="[15.0.0]" />
  <PackageVersion Include="MSTest.TestAdapter" Version="[1.1.18]" />
  <PackageVersion Include="MSTest.TestFramework" Version="[1.1.18]" />
  <PackageVersion Include="NETStandard.Library" Version="[1.6.1]" />
  <PackageVersion Include="Microsoft.NETCore.App" Version="[1.1.2]" />
  <PackageVersion Include="Newtonsoft.Json" Version="[9.0.1]" />
</ItemGroup>
```

During project evaluation, the original list of `PackageReference` items is replaced with a package containing a version.  This allows existing tooling to still properly read the package references so command-line restore, Visual Studio restore, and other functionality continues to work.  

There is an MSBuild target that runs before build which detects if any packages are referenced in a project that do not have a version specified in the central file.  This ensures that all package versions are defined centrally and users can't introduce bad package references.

```
Build FAILED.

"D:\CentralPackageVersions\src\UnitTestProject\UnitTestProject.csproj" (default target) (1) ->
(ApplyPackageVersions target) ->
  D:\CentralPackageVersions\src\UnitTestProject\UnitTestProject.csproj : error : The package reference
  'Foo' must have a version defined in 'D:\CentralPackageVersions\PackageVersions.props'.
```

## What doesn't work
At this time, updating a package reference in Visual Studio will add a version to the project rather than updating the central file.  Building the project will generate an error that the version should be in the central file and so users must manually move the version from the project to the central file.
