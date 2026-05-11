# .NET MSBuild SDK-Konventionen — Spickzettel

Reine Referenz: Verbatim-Quotes aus offizieller Microsoft-/dotnet-Dokumentation für die in einem typischen .NET-SDK-Repo einschlägigen MSBuild-Dateien. Keine eigene Einschätzung.

---

## Teil A — Dateien, die MSBuild per Konvention selbst kennt

Diese Dateinamen sind in Microsoft-Doku belegt. Für jede Datei: was die Doku tatsächlich sagt.

### `Directory.Build.props`

> "*Microsoft.Common.props* searches your directory structure for the *Directory.Build.props* file. If it finds one, it imports the file and reads the properties defined within it." — [[1]](#q1)

> "*Directory.Build.props* is imported early in the sequence of imported files, which can be important if you need to set a property that is used by imports, especially those that are implicitly imported by using the `Sdk` attribute, such as when using the .NET SDK in most .NET project files." — [[1]](#q1)

### `Directory.Build.targets`

> "Similarly, *Microsoft.Common.targets* looks for *Directory.Build.targets*." — [[1]](#q1)

> "*Directory.Build.targets* is imported from *Microsoft.Common.targets* after importing `.targets` files from NuGet packages. So, it can override properties and targets defined in most of the build logic, or set properties for all your projects regardless of what the individual projects set." — [[1]](#q1)

### `Directory.Packages.props`

> "To get started with Central Package Management, create a `Directory.Packages.props` file at the root of your repository and set the MSBuild property `ManagePackageVersionsCentrally` to `true`." — [[7]](#q7)

> "Inside `Directory.Packages.props`, define `<PackageVersion />` elements to specify the package IDs and versions used by your projects." — [[7]](#q7)

### `.editorconfig`

> "EditorConfig files are used to provide options that apply to specific source files or folders. Options are placed under section headers to identify the applicable files and folders." — [[9]](#q9)

> "the compiler implicitly includes *.editorconfig* files in a project" — [[9b]](#q9b)

### `BannedSymbols.txt` / `BannedSymbols.*.txt`

> Aus der `BannedApiAnalyzers.Help.md`: die Datei(en) `BannedSymbols.txt` bzw. `BannedSymbols.*.txt` werden über `AdditionalFiles` ins Projekt eingebunden, damit der Analyzer sie liest. — [[14]](#q14)

> XML-Form laut Doku:
> ```xml
> <ItemGroup>
>   <AdditionalFiles Include="BannedSymbols.txt" />
> </ItemGroup>
> ```
> — [[14]](#q14)

---

## Teil B — Repo-spezifische Dateinamen

`Build/Common/*` und `Build/Enforcement/*` sind **keine** offiziellen MSBuild-Konventionen. MSBuild kennt nur die `Directory.*`-Dateien aus Teil A implizit. Was unter `Build/...` liegt, wird in deinem Repo per `<Import Project="…" />` aus den `Directory.Build.*`-Dateien eingebunden.

Es gibt also keine Microsoft-Doku zu diesen Dateinamen. Belegbar ist nur, was die typischerweise dort gesetzten **Properties** offiziell bedeuten.

### `Build/Common/Common.props` — typische Properties

| Property | Verbatim aus MS-Doku | Quelle |
| --- | --- | --- |
| `LangVersion` | "The **LangVersion** option causes the compiler to accept only syntax that is included in the specified C# language specification" | [[3]](#q3) |
| `Nullable` | "The **Nullable** option lets you specify the nullable context. […] The argument must be one of `enable`, `disable`, `warnings`, or `annotations`." | [[4]](#q4) |
| `TreatWarningsAsErrors` | "The **TreatWarningsAsErrors** option treats all warnings as errors." | [[5]](#q5) |

### `Build/Common/GlobalPackages.props` / `Build/Common/Version.props`

| Property | Verbatim aus MS-Doku | Quelle |
| --- | --- | --- |
| `ManagePackageVersionsCentrally` | "set the MSBuild property `ManagePackageVersionsCentrally` to `true`" | [[7]](#q7) |
| `<PackageVersion />` Item | "define `<PackageVersion />` elements to specify the package IDs and versions used by your projects" | [[7]](#q7) |
| `Version` / `VersionPrefix` / `VersionSuffix` / `PackageVersion` | Pack-Target-Tabelle: `Version` → `PackageVersion` (Default = `Version`); `VersionPrefix` / `VersionSuffix` separat überschreibbar | [[8]](#q8) |

### `Build/Enforcement/DeterminismAndSourceLink.props`

| Property | Verbatim aus MS-Doku | Quelle |
| --- | --- | --- |
| `Deterministic` | "Causes the compiler to produce an assembly whose byte-for-byte output is identical across compilations for identical inputs." | [[10]](#q10) |
| `Deterministic`-Default | "For modern .NET projects, deterministic compilation is enabled by default (the `Deterministic` property defaults to `true`)." | [[10]](#q10) |
| SourceLink | "Source Link is a technology that enables source code debugging of .NET assemblies from NuGet by developers. Source Link executes when creating the NuGet package and embeds source control metadata inside assemblies and the package." | [[11]](#q11) |

### `Build/Common/ContinuousIntegrationBuild.props`

> "The `ContinuousIntegrationBuild` property indicates whether a build is executing on a continuous integration (CI) server. When set to `true`, this property enables settings that only apply to official builds as opposed to local builds on a developer machine. For example, stored file paths are normalized for official builds." — [[12]](#q12)

> Beispiel aus der Doku für GitHub Actions:
> ```xml
> <PropertyGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
>   <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
> </PropertyGroup>
> ```
> — [[12]](#q12)

### `Build/Common/SourceGenerators.targets`

> "Source generators aim to enable *compile time metaprogramming*, that is, code that can be created at compile time and added to the compilation. Source generators are able to read the contents of the compilation before running, as well as access any *additional files*." — [[6]](#q6)

### `Build/Common/Tests.targets`, `Build/Common/Npm.targets`, `Build/Common/Common.targets`, `Build/Enforcement/Enforcement.props`, `Build/Enforcement/VersionEnforcement.targets`

Keine Microsoft-Doku zu diesen Dateinamen. Inhalt ist Repo-Konvention. Was `<Import>` und `Directory.Build.targets`-Mechanik betrifft, gilt Teil A [[1]](#q1).

---

## Teil C — Konventionsrelevante Begleit­dateien

| Datei | Verbatim aus MS-Doku | Quelle |
| --- | --- | --- |
| `Global.editorconfig`, `CodingStyle.editorconfig` | "EditorConfig files are used to provide options that apply to specific source files or folders." | [[9]](#q9) |
| `Compiler.editorconfig`, `Analyzer.*.editorconfig` | "Code analysis rules have various configuration options. You specify these options as key-value pairs in one of the following analyzer configuration files: EditorConfig file […]; Global AnalyzerConfig file […]" | [[9]](#q9) |

---

## Praktisch prüfen — direkt aus der Doku

> "`-preprocess[:{filepath}]` `-pp[:{filepath}]` — Create a single, aggregated project file by inlining all the files that would be imported during a build, with their boundaries marked. You can use this switch to more easily determine which files are being imported, from where the files are being imported, and which files contribute to the build." — [[15]](#q15)

```bash
dotnet msbuild <Projekt>.csproj -pp:preprocessed.xml
```

Dann im Ergebnis nach den interessanten Imports / Properties suchen:

```bash
rg "TreatWarningsAsErrors|Nullable|LangVersion|Deterministic|ContinuousIntegrationBuild|SourceLink|PackageVersion|ManagePackageVersionsCentrally" preprocessed.xml
```

---

## Quellenverzeichnis

<a id="q1"></a>**[1]** Microsoft Learn — *Customize the build by folder* (`Directory.Build.props` / `Directory.Build.targets`, Import-Reihenfolge).
<https://learn.microsoft.com/visualstudio/msbuild/customize-by-directory>

<a id="q2"></a>**[2]** Microsoft Learn — *How MSBuild builds projects → User-configurable imports*.
<https://learn.microsoft.com/visualstudio/msbuild/build-process-overview>

<a id="q3"></a>**[3]** Microsoft Learn — *C# Compiler Options for language feature rules → `LangVersion`*.
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/language#langversion>

<a id="q4"></a>**[4]** Microsoft Learn — *C# Compiler Options for language feature rules → `Nullable`*.
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/language#nullable>

<a id="q5"></a>**[5]** Microsoft Learn — *C# Compiler Options to report errors and warnings → `TreatWarningsAsErrors`*.
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/errors-warnings#treatwarningsaserrors>

<a id="q6"></a>**[6]** Microsoft Learn — *.NET Compiler Platform SDK → Source generators*.
<https://learn.microsoft.com/dotnet/csharp/roslyn-sdk/#source-generators>

<a id="q7"></a>**[7]** Microsoft Learn — *Central Package Management (CPM)* (`Directory.Packages.props`, `ManagePackageVersionsCentrally`, `<PackageVersion />`).
<https://learn.microsoft.com/nuget/consume-packages/central-package-management>

<a id="q8"></a>**[8]** NuGet Docs — *Pack target* (Tabelle: `Version` / `PackageVersion` / `VersionPrefix` / `VersionSuffix`) + Microsoft Learn — *MSBuild reference for .NET SDK projects*.
<https://learn.microsoft.com/nuget/reference/msbuild-targets#pack-target> · <https://learn.microsoft.com/dotnet/core/project-sdk/msbuild-props>

<a id="q9"></a>**[9]** Microsoft Learn — *Configuration files for code analysis rules* (EditorConfig + Global AnalyzerConfig, Severity, Precedence).
<https://learn.microsoft.com/dotnet/fundamentals/code-analysis/configuration-files>

<a id="q9b"></a>**[9b]** Microsoft Learn — *EditorConfig files implicitly included* (.NET 6+ Compiler-Verhalten).
<https://learn.microsoft.com/dotnet/core/compatibility/sdk/6.0/editorconfig-additional-files>

<a id="q10"></a>**[10]** Microsoft Learn — *C# Compiler Options that control code generation → `Deterministic`*.
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/code-generation#deterministic>

<a id="q11"></a>**[11]** Microsoft Learn — *Source Link* (Library Guidance) + Repo `dotnet/sourcelink`.
<https://learn.microsoft.com/dotnet/standard/library-guidance/sourcelink> · <https://github.com/dotnet/sourcelink>

<a id="q12"></a>**[12]** Microsoft Learn — *MSBuild reference for .NET SDK projects → `ContinuousIntegrationBuild`* (mit `TF_BUILD`- und `GITHUB_ACTIONS`-Beispielen).
<https://learn.microsoft.com/dotnet/core/project-sdk/msbuild-props#continuousintegrationbuild>

<a id="q13"></a>**[13]** Microsoft Learn — *Define consistent coding styles with EditorConfig*.
<https://learn.microsoft.com/visualstudio/ide/create-portable-custom-editor-options>

<a id="q14"></a>**[14]** dotnet/roslyn-analyzers — *Microsoft.CodeAnalysis.BannedApiAnalyzers — BannedApiAnalyzers.Help.md*.
<https://github.com/dotnet/roslyn-analyzers/blob/main/src/Microsoft.CodeAnalysis.BannedApiAnalyzers/BannedApiAnalyzers.Help.md>

<a id="q15"></a>**[15]** Microsoft Learn — *MSBuild command-line reference → `-preprocess` / `-pp` Switch*.
<https://learn.microsoft.com/visualstudio/msbuild/msbuild-command-line-reference#switches>

---

## Lizenz

[CC0-1.0](./LICENSE) — Public Domain. Inhalt darf frei kopiert, geändert und weiterverwendet werden.

Verbatim-Zitate gehören Microsoft Corporation bzw. den jeweiligen Rechteinhabern und werden nach Fair-Use-/Zitatrecht referenziert.
