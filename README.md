# S4Forge.SDK

The S4Forge.SDK is a C# library/MSBuild helper designed to facilitate the development of modules (mods) that interact with S4Forge.
It provides a set of APIs and tools to streamline the process of building, testing, and deploying modules.

## MSBuild Properties and Item Groups
When building a module using S4Forge.SDK, you can define various MSBuild properties and item groups in your project file to customize the build process and manifest generation.

### Manifest

Available manifest MSBuild properties:

> Entries marked with * are optional and have default values.

| Property | Description |
| -- | -- |
| `ManifestId` | The id of the resource - needs to be unique for the type of resource and shared across all version. Shouldn't include any white space character |
| `ManifestName` | The name of the resource |
| `ManifestVersion`* | SemVer of the resource, defaults to $(AssemblyVersion) |
| `ManifestType`* | What type of resource this is, e.g. plugin, engine, base, etc. defaults to $(ProjectType) _see later_ |
| `ManifestAssetUrl`* | Url to download release output assets, e.g. gitlab release url. Defaults to `$(RepositoryUrl)/releases/latest/download/$(ReleaseFile)` or `$(ReleaseOutputPath)` when $(RepositoryUrl) is not set|
| `ManifestClearResidualFiles`* | Whether to delete any files in the output folder (be careful to not set to true, when it's unclear what files should be kept) |
| `ManifestEmbedded`* | Whether this manifest file should be embedded in the assembly or not. Defaults to false |
| `ManifestEntryPoint`* | Path of assembly that contains your e.g. IModule class implementations, defaults to $(TargetFileName) |
| `ManifestLibraryFolder`* | Folder where other external library files are stored that need to be loaded during initialization |


Available manifest MSBuild item groups:

| Item Group | Description |   
|--|--| 
| `ManifestIgnoredEntries`* | Files to be ignored when updating/clearing residual files. Could be used to keep logs, settings, etc.
| `ManifestRelationships`* | Dependency of this manifest - include the id of the dependency |
| > `Manifest` | Url to the newest manifest (e.g. latest release on gitlab) |
| > `Optional` | Whether this dependency is optional |
| > `Minimum` | Minimum supported version required to work - can have wildcards |
| > `Maximum` | Maximum \^-^ |
| > `Verified` | Verified \^-^ |

### Release Packaging

| Property | Description |
| -- | -- |
| `ExportToS4`* | Whether to export the build output to the S4Forge game folder after a successful build. Defaults to true for local builds. |
| `ProjectType` | The type of project being built. Used to determine the export path |
| > `ProjectType=Module` | Exports to the Modules folder in the Plugins directory of Settlers 4, with your module in it's own folder. (plugins/Modules/$(TargetName) |
| > `ProjectType=Engine` | Exports directly to the Engines folder in the Forge directory. (plugins/Forge/Engines/ |
| > `ProjectType=Bootstrap` | Reserved for the main project. Exports to the Plugins folder directly |
| > `ProjectType=Forge` | Reserved for the main project. Exports to the Forge folder |


| ItemGroup | Description |
|--|--|
| `ExportFile`* | Defines the files that should be exported to the game folder. |
| > `ExportFileDir` | Directory relative to the chosen ExportPath based on the ProjectType. E.g. $(ModuleExportPath)/$(ExportFileDir) |

## New MSBuild Targets

The S4Forge.SDK provides several MSBuild targets that can be invoked during the build process to perform specific tasks related to module development.

| Target | Description |
|--|--|
| `GenerateManifest` | Generates the manifest file based on the defined MSBuild properties and item groups. Is automatically called after `PreBuildEvent` |
| `ExportToS4` | Exports the build output to the S4Forge game folder based on the defined export properties and item groups. Is automatically called after `Build` |
| `Release` | Packages the build output and manifest into a zip file for release. Can be called manually after `Build` |

## Example

Here is an example of how to use S4Forge.SDK configured in an external .props file (This example is taken from the DebugModule):
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <!-- Forge Build System: -->
    <PropertyGroup>
        <ProjectType>Module</ProjectType>
    </PropertyGroup>

    <!-- Manifest: -->
    <PropertyGroup>
        <ManifestId>debug-menu</ManifestId>
        <ManifestName>Debug Menu</ManifestName>
        <ManifestAssetUrl>Url to download release output assets, e.g. gitlab release url</ManifestAssetUrl>
        <ManifestClearResidualFiles>true</ManifestClearResidualFiles>
    </PropertyGroup>
</Project>
```

## GitHub Workflows
This repository also contains prepared GitHub workflows that should help with how to build and release modules.

There are two workflows available:

### publish-workflow.yml
This workflow builds and published the project to NuGet. This is mostly only useful for engines and modules that can be referenced by other modules.

| Parameter | Description | Default |
| -- | -- | -- |
| version | The version to publish | ${{github.ref_name}} |
| package_folder | The folder where the .csproj/.nuspec file is located. Should not have a trailing slash | . |
| package_path | The path to the .nupkg file to publish. If not set, it will be determined based on the package_name and version |  |
| package_name | The name of the package to publish |  |
| nuget_repo | The NuGet repository to publish to | https://api.nuget.org/v3/index.json |
| nuget_key | The NuGet API key to use for publishing |  |


### release-workflow.yml
This workflow builds the project, creates a release package and publishes it as a GitHub release. It uses the `Release` MSBuild target to create the release package. The GitHub release is staged as a draft for review before publishing.

| Parameter | Description | Default |
| -- | -- | -- |
| version | The version to publish | ${{github.ref_name}} |
| package_folder | The folder where the .csproj/.nuspec file is located | . |
| multi-stage | If true, the release will be created in multiple stages e.g. the release will be created as draft with the version number and under a "latest" tag | true |
| SENTRY_KEY | The Sentry key to use for uploading symbols |  |

