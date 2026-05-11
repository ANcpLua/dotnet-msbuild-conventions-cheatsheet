# .NET MSBuild SDK-Konventionen — Spickzettel

> Welche MSBuild-Dateien in einem .NET-SDK-Repo typischerweise Konventionen erzwingen — mit Quellenangaben aus offizieller Microsoft- und dotnet-Dokumentation.

---

## TL;DR

Für SDK-Konventionen ist **`Directory.Build.targets`** der wahrscheinlichste globale Einstieg (MSBuild importiert die Datei automatisch [[1]](#q1)). Die eigentliche Policy steckt aber meist in den Repo-eigenen Konventionsordnern **`Build/Common/*`** und **`Build/Enforcement/*`**, die per `Import`-Element von dort eingebunden werden.

> ⚠️ Dateinamen wie `Common.props`, `Enforcement.props`, `VersionEnforcement.targets` sind **Repo-Konventionen**, keine MSBuild-Standards. MSBuild kennt nur die `Directory.*`-Dateien implizit [[1]](#q1) [[2]](#q2). Alles andere ist `<Import Project="…" />`.

---

## 1) MSBuild-Dateien, die wahrscheinlich .NET-/SDK-Konventionen erzwingen

| Datei | Wahrscheinliche Rolle | Quelle |
| --- | --- | --- |
| **`Directory.Build.targets`** | Globaler Hook für alle Projekte im Repo. Sehr wahrscheinlich zentral für Enforcement, weil MSBuild diese Datei automatisch importiert (spät, *nach* dem Projekt — kann also Properties aus Projekt + NuGet-Paketen überschreiben). | [[1]](#q1) [[2]](#q2) |
| **`Build/Common/Common.props`** | Gemeinsame Properties: `TargetFramework`, `LangVersion`, `Nullable`, `TreatWarningsAsErrors`, Paket-/Analyzer-Einstellungen. | [[3]](#q3) [[4]](#q4) [[5]](#q5) |
| **`Build/Common/Common.targets`** | Gemeinsame Targets: Validierungen, Checks, Build-Konventionen nach Projektimport. | [[1]](#q1) |
| **`Build/Common/Tests.targets`** | Konventionen speziell für Testprojekte. | [[2]](#q2) |
| **`Build/Common/SourceGenerators.targets`** | Regeln/Konventionen für Source-Generator-Projekte (z. B. `IsRoslynComponent`, Analyzer-Packaging). | [[6]](#q6) |
| **`Build/Common/Npm.targets`** | Nur relevant, falls SDK/Web-Projekte Node/npm-Buildschritte erzwingen. | [[2]](#q2) |
| **`Build/Common/GlobalPackages.props`** | Zentralisierte Paketversionen (CPM via `Directory.Packages.props` / `ManagePackageVersionsCentrally`). Kann indirekt Konventionen erzwingen. | [[7]](#q7) |
| **`Build/Common/Version.props`** | Versionierungs-Konventionen (`Version`, `VersionPrefix`, `AssemblyVersion`, …). | [[8]](#q8) |
| **`Build/Enforcement/Enforcement.props`** | Sehr wahrscheinlich explizite Enforcement-Konfiguration (Analyzer-Severity, `EnforceCodeStyleInBuild`, …). | [[1]](#q1) [[9]](#q9) |
| **`Build/Enforcement/VersionEnforcement.targets`** | Version-/Package-/Assembly-Version-Regeln. | [[8]](#q8) |
| **`Build/Enforcement/DeterminismAndSourceLink.props`** | Erzwingt deterministische Builds (`<Deterministic>true</Deterministic>`) und SourceLink-Konventionen. | [[10]](#q10) [[11]](#q11) |
| **`Build/Common/ContinuousIntegrationBuild.props`** | CI-spezifische Regeln, typischerweise `ContinuousIntegrationBuild=true`, deterministische Outputs, SourceLink/PathMap-Verhalten. | [[12]](#q12) |

## 2) Konventionsrelevant, aber nicht primär MSBuild

| Datei | Rolle | Quelle |
| --- | --- | --- |
| **`Global.editorconfig`** | Code-Style/Analyzer-Regeln global (Naming, Format, Severity). | [[9]](#q9) [[13]](#q13) |
| **`Compiler.editorconfig`** | Compiler-/Analyzer-Regeln (z. B. `dotnet_diagnostic.CSxxxx.severity`). | [[9]](#q9) |
| **`Analyzer.*.editorconfig`** | Analyzer-spezifische Regeln (per-Kategorie / per-Rule). | [[9]](#q9) |
| **`CodingStyle.editorconfig`** | Style-Konventionen (Layout, Naming, IDE-Style-Regeln). | [[13]](#q13) |
| **`BannedSymbols.NewtonsoftJson.txt`** | Verbietet bestimmte APIs/Symbole via `Microsoft.CodeAnalysis.BannedApiAnalyzers`. Wird als `<AdditionalFiles Include="BannedSymbols.txt" />` (oder `BannedSymbols.*.txt`) eingebunden. | [[14]](#q14) |

---

## 3) Wichtigste Einstiegspunkte

```text
Directory.Build.targets
Build/Common/Common.props
Build/Common/Common.targets
Build/Enforcement/Enforcement.props
Build/Enforcement/VersionEnforcement.targets
Build/Enforcement/DeterminismAndSourceLink.props
Build/Common/ContinuousIntegrationBuild.props
```

Importreihenfolge laut MSBuild [[1]](#q1) [[2]](#q2):

1. **früh:** `Directory.Build.props` (via `Microsoft.Common.props`) → setzt Defaults, bevor das Projekt evaluiert wird.
2. **Projekt-Body** + alle `*.props` aus NuGet-Paketen.
3. **`*.targets` aus NuGet-Paketen.**
4. **spät:** `Directory.Build.targets` (via `Microsoft.Common.targets`) → kann Paket-/Projekt-Properties überschreiben.

## 4) Praktisch prüfen

Vollen Import-Graph als „präprozessiertes" XML dumpen [[15]](#q15):

```bash
dotnet msbuild ANcplua.NET.Sdk.csproj -pp:preprocessed.xml
```

Dann in `preprocessed.xml` nach diesen Imports suchen:

```text
Directory.Build.targets
Common.props
Common.targets
Enforcement.props
VersionEnforcement.targets
DeterminismAndSourceLink.props
ContinuousIntegrationBuild.props
```

## 5) Konkrete Properties / Targets suchen

Wenn du wissen willst, **welche Datei eine bestimmte Konvention setzt**, suche im Repo nach den Properties/Targets:

```bash
rg "TreatWarningsAsErrors|Nullable|LangVersion|Enforce|Deterministic|ContinuousIntegrationBuild|SourceLink|PackageVersion|NoWarn|WarningsAsErrors"
```

Häufige Treffer (Property → wahrscheinliche Datei):

| Property | Wo gesetzt | Quelle |
| --- | --- | --- |
| `TreatWarningsAsErrors` | `Common.props` / `Enforcement.props` | [[5]](#q5) |
| `Nullable` | `Common.props` | [[4]](#q4) |
| `LangVersion` | `Common.props` | [[3]](#q3) |
| `Deterministic` | `DeterminismAndSourceLink.props` (default in modernen SDKs: `true`) | [[10]](#q10) |
| `ContinuousIntegrationBuild` | `ContinuousIntegrationBuild.props` (per CI-Variable wie `TF_BUILD`, `GITHUB_ACTIONS` bedingt) | [[12]](#q12) |
| `ManagePackageVersionsCentrally` + `<PackageVersion .../>` | `Directory.Packages.props` / `GlobalPackages.props` | [[7]](#q7) |

---

## 6) Einschätzung

Für SDK-Konventionen ist **`Directory.Build.targets`** der wahrscheinlichste globale Einstieg, aber die eigentliche Policy steckt vermutlich in **`Build/Common/*`** und **`Build/Enforcement/*`**. Das deckt sich mit der offiziellen Empfehlung, projektübergreifende Build-Logik in `Directory.Build.targets` zu sammeln und projektweite Properties über importierte `.props`-Dateien zu konsolidieren [[1]](#q1).

---

## Quellenverzeichnis

<a id="q1"></a>**[1]** Microsoft Learn — *Customize the build by folder* (`Directory.Build.props` / `Directory.Build.targets`, Import-Reihenfolge, Search-Scope).
<https://learn.microsoft.com/visualstudio/msbuild/customize-by-directory>

<a id="q2"></a>**[2]** Microsoft Learn — *How MSBuild builds projects → User-configurable imports*.
<https://learn.microsoft.com/visualstudio/msbuild/build-process-overview>

<a id="q3"></a>**[3]** Microsoft Learn — *C# Compiler Options for language feature rules → `LangVersion`*.
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/language#langversion>

<a id="q4"></a>**[4]** Microsoft Learn — *C# Compiler Options for language feature rules → `Nullable`*.
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/language#nullable>

<a id="q5"></a>**[5]** Microsoft Learn — *C# Compiler Options to report errors and warnings → `TreatWarningsAsErrors`*.
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/errors-warnings#treatwarningsaserrors>

<a id="q6"></a>**[6]** Microsoft Learn — *.NET Compiler Platform SDK → Source generators* (inkrementelle Generatoren, `IIncrementalGenerator`).
<https://learn.microsoft.com/dotnet/csharp/roslyn-sdk/#source-generators>

<a id="q7"></a>**[7]** Microsoft Learn — *Central Package Management (CPM)* (`Directory.Packages.props`, `ManagePackageVersionsCentrally`, `GlobalPackageReference`).
<https://learn.microsoft.com/nuget/consume-packages/central-package-management>

<a id="q8"></a>**[8]** NuGet Docs — *Pack target* (kanonische Tabelle für `Version` / `PackageVersion` / `VersionPrefix` / `VersionSuffix` als MSBuild-Properties) + Microsoft Learn — *MSBuild reference for .NET SDK projects* (Top-Level-Referenz inkl. Assembly-Info- und Package-Properties-Sektionen).
<https://learn.microsoft.com/nuget/reference/msbuild-targets#pack-target> · <https://learn.microsoft.com/dotnet/core/project-sdk/msbuild-props>

<a id="q9"></a>**[9]** Microsoft Learn — *Configuration files for code analysis rules* (EditorConfig + Global AnalyzerConfig, Severity, Precedence).
<https://learn.microsoft.com/dotnet/fundamentals/code-analysis/configuration-files>

<a id="q10"></a>**[10]** Microsoft Learn — *C# Compiler Options that control code generation → `Deterministic`* (default `true` in modernen .NET-Projekten).
<https://learn.microsoft.com/dotnet/csharp/language-reference/compiler-options/code-generation#deterministic>

<a id="q11"></a>**[11]** Microsoft Learn — *Source Link* (Library Guidance, deterministische Builds, Symbolpakete) und Repo `dotnet/sourcelink`.
<https://learn.microsoft.com/dotnet/standard/library-guidance/sourcelink> · <https://github.com/dotnet/sourcelink>

<a id="q12"></a>**[12]** Microsoft Learn — *MSBuild reference for .NET SDK projects → `ContinuousIntegrationBuild`* (mit Beispielen für `TF_BUILD` und `GITHUB_ACTIONS`).
<https://learn.microsoft.com/dotnet/core/project-sdk/msbuild-props#continuousintegrationbuild>

<a id="q13"></a>**[13]** Microsoft Learn — *Define consistent coding styles with EditorConfig*.
<https://learn.microsoft.com/visualstudio/ide/create-portable-custom-editor-options>

<a id="q14"></a>**[14]** dotnet/roslyn-analyzers — *Microsoft.CodeAnalysis.BannedApiAnalyzers* (`BannedSymbols.txt`, `BannedSymbols.*.txt` via `<AdditionalFiles>`).
<https://github.com/dotnet/roslyn-analyzers/blob/main/src/Microsoft.CodeAnalysis.BannedApiAnalyzers/BannedApiAnalyzers.Help.md>

<a id="q15"></a>**[15]** Microsoft Learn — *MSBuild command-line reference* (`-preprocess` / `-pp` Switch).
<https://learn.microsoft.com/visualstudio/msbuild/msbuild-command-line-reference>

---

## Lizenz

[CC0-1.0](./LICENSE) — Public Domain. Inhalt darf frei kopiert, geändert und weiterverwendet werden.

Original-Quellen sind Eigentum ihrer jeweiligen Rechteinhaber (Microsoft Learn, dotnet/* Repos) und nur referenziert, nicht reproduziert.
