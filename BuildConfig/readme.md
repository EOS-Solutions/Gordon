# Build Configurations

Build configurations are JSON files that control how an AL project is compiled. They let you define compiler options, file filters, preprocessor symbols, version metadata, and more — without touching your `app.json` or the build tool's command line.

---

## Folder Layout

Build configuration files live inside a special subfolder named **`.buildconfig`**, which can be placed at any level above your project folder. Configurations at all levels—from the project folder itself up through any parent directories to the workspace root—are collected and merged.

```
MyWorkspace/
├── .buildconfig/          ← workspace-level configurations
│   ├── .global.json
│   ├── default.json
│   └── release.json
│
├── MyProject/
│   ├── app.json
│   └── .buildconfig/      ← project-level configurations
│       ├── default.json
│       └── release.json
│
└── AnotherProject/
    └── app.json           ← no .buildconfig here; falls back to workspace level
```

- Every configuration file uses the `.json` extension.
- The **name** of a configuration is the filename without the extension (e.g., `release.json` → `release`).
- A file named **`.global.json`** in any `.buildconfig` folder is always included and applied to every configuration at that level, regardless of which named configuration is loaded.

---

## Named Configurations

You can have as many named configurations as you need. Common examples:

| File | Configuration name |
|------|--------------------|
| `default.json` | *(default, no name)* |
| `release.json` | `release` |
| `debug.json` | `debug` |
| `test.json` | `test` |

When a build is triggered with a specific configuration name (e.g., `release`), the tool loads `release.json`. If a file with that name does not exist in the project's `.buildconfig` folder, the tool automatically falls back to `default.json` at the same level before looking in parent folders.

---

## The `.global.json` File

A file named **`.global.json`** in any `.buildconfig` folder is special: its settings are **always applied**, regardless of which named configuration is being loaded.

Use `.global.json` to set workspace-wide or project-wide defaults that should hold for every build — for example, a shared output folder, a default ruleset, or a common set of preprocessor symbols.

```
.buildconfig/
├── .global.json    ← always applied at this level
├── default.json
└── release.json
```

> `.global.json` is **not** listed when enumerating available configuration names as it is not really a configuration by itself.

---

## Inheritance and Recursion

When a configuration is loaded for a project folder, the tool **walks upward** through the directory tree, collecting configurations from every `.buildconfig` folder it finds along the way. Configuration files can be placed at any level above the current folder and will be included in the merge. This means a configuration defined at a higher level (e.g., workspace root) acts as a **base**, and deeper-level files (e.g., inside the project folder) **override** it.

### How the merge works

1. The tool starts at the project's `.buildconfig` folder and tries to read the requested configuration file (e.g., `release.json`), falling back to `default.json` if not found.
2. If a `.global.json` exists at that level, it is collected as well.
3. The tool then moves to the parent folder's `.buildconfig` and repeats the process.
4. This continues until the root of the directory hierarchy is reached.
5. All collected files are merged from **outermost (highest) level inward**: a value set at the project level always wins over the same value set at a workspace or parent level.

### Merge rules by property type

| Property type | Merge behaviour |
|---------------|-----------------|
| Scalar (`string`, `bool`, `Version`, …) | The **innermost** (most specific) non-null value is used. |
| List (`includePatterns`, `excludePatterns`, `preprocessorSymbols`) | Values from **all levels are combined**. |

### Example

```
Workspace/
├── .buildconfig/
│   ├── .global.json      → sets warnAsError: true
│   └── release.json      → sets outputFolder: "out/release"
│
└── MyProject/
    └── .buildconfig/
        ├── .global.json  → sets preprocessorSymbols: ["MYFEATURE"]
        └── release.json  → sets metadata.version: "2.0.0.0"
```

Loading `release` for `MyProject` produces a merged configuration with:
- `WarnAsError: true` — from workspace `.global.json`
- `OutputFolder: "out/release"` — from workspace `release.json`
- `PreprocessorSymbols: ["MYFEATURE"]` — from project `.global.json`
- `Metadata.Version: "2.0.0.0"` — from project `release.json`

---

## Configuration Options Reference

A configuration file is a JSON object. All properties are optional.

### Top-level properties

| Property | Type | Description |
|----------|------|-------------|
| `Description` | `string` | A human-readable label for this configuration. |
| `Metadata` | object | Version and source-control metadata to stamp into the app before building. See [Metadata](#metadata). |
| `Options` | object | Compiler (`alc.exe`) options. See [Options](#options). |
| `OptionsFile` | `string` | Path to a JSON file containing compiler options. When specified, `Options` is ignored. |
| `IncludePatterns` | `string[]` | Glob patterns of files to include in the build. When omitted, all files are included. |
| `ExcludePatterns` | `string[]` | Glob patterns of files to exclude after include patterns are applied. |
| `PreprocessorSymbols` | `string[]` | Additional preprocessor symbols to define during compilation. |
| `IncludeTestDependencies` | `bool` | `true` to add test dependencies, `false` to remove them, omit to leave as-is. |
| `KeepBuildFolder` | `bool` | When `true`, the temporary build folder is not deleted after the build completes. Useful for debugging. |

### Metadata

Nested under the `metadata` key.

| Property | Type | Description |
|----------|------|-------------|
| `Version` | `string` | Sets the app version (e.g. `"2.1.0.0"`) before building. |
| `IncrementBuildVersion` | `bool` | When `true`, the build (fourth) component of the version is incremented automatically and written back to the source file. See [Incrementing Build Version](#incrementing-build-version). |
| `RepositoryUrl` | `string` | Source repository URL to embed in the package (runtime ≥ 12). |
| `Commit` | `string` | Source commit SHA to embed in the package (runtime ≥ 12). |
| `BuildBy` | `string` | Name of the system or agent that performed the build (runtime ≥ 12). |
| `BuildUrl` | `string` | URL of the build job or pipeline run (runtime ≥ 12). |

### Options

Nested under the `options` key. These map directly to `alc.exe` arguments.

| Property | Type | Description |
|----------|------|-------------|
| `AlcFolder` | `string` | Path to a specific `alc.exe` folder. Defaults to the most recent AL Language extension. |
| `OutputFilename` | `string` | Full path for the output `.app` file. Takes precedence over `OutputFolder`. |
| `OutputFolder` | `string` | Folder where the output `.app` file is written. Ignored if `OutputFilename` is set. |
| `ErrorLog` | `string` | File path where compiler diagnostics are written. |
| `PackageCacheFolders` | `string[]` | Folders containing `.app` symbol files used during compilation. |
| `AssemblyProbingPaths` | `string[]` | Additional .NET assembly search paths. |
| `WarnAsError` | `bool` | Treats all compiler warnings as errors. |
| `RuleSet` | `string` | Path to a `.ruleset` file that overrides diagnostic severities. |
| `Target` | `string` | Compilation target: `internal`, `solution`, or `extension`. |
| `GenerateCode` | `bool` | Include metadata and IL code in the output package. |
| `GenerateReportLayout` | `bool` | Generate or update report layouts from the dataset. |
| `Analyzers` | `string[]` | Paths to additional code-analysis DLLs. |
| `ParallelDegree` | `int` | Maximum number of parallel compiler tasks. |
| `ReportSuppressedDiagnostics` | `bool` | Emit diagnostics that are suppressed in source code. |
| `SourceRepositoryUrl` | `string` | Repository URL passed to the compiler (separate from `Metadata.RepositoryUrl`). |
| `SourceCommit` | `string` | Commit SHA passed to the compiler. |
| `BuildBy` | `string` | Build agent name passed to the compiler. |
| `BuildUrl` | `string` | Build URL passed to the compiler. |

---

## Minimal Examples

**`default.json`** — baseline settings used by every build:
```json
{
  "Options": {
    "OutputFolder": "out",
    "WarnAsError": true
  }
}
```

**`release.json`** — override for release builds:
```json
{
  "Description": "Production release build",
  "Metadata": {
    "Version": "3.0.0.0",
    "IncrementBuildVersion": true
  },
  "Options": {
    "OutputFolder": "out/release",
    "Target": "extension"
  },
  "PreprocessorSymbols": ["RELEASE"]
}
```

**`.global.json`** — settings applied to every named configuration at this level:
```json
{
  "Options": {
    "PackageCacheFolders": ["../../.alpackages"]
  }
}
```
---

## Incrementing Build Version

When `Metadata.IncrementBuildVersion` is set to `true`, the build process automatically increments the **revision component** (the fourth number) of the application version.

### How it works

1. Before building, the configuration is evaluated.
2. If both `IncrementBuildVersion: true` and a `Version` are specified, the revision number is incremented (e.g., `3.0.0.5` → `3.0.0.6`).
3. The updated version is automatically written back to the configuration file that originally specified the version.
4. On subsequent builds with the same configuration, the version continues to increment from where it left off.
5. If `IncrementBuildVersion: true` but **no `Version` is specified** in any configuration file (including inherited ones), the increment is silently skipped—no error is raised and the app version remains unchanged.

### Example

Your `release.json` contains:
```json
{
  "Metadata": {
    "Version": "3.0.0.5",
    "IncrementBuildVersion": true
  }
}
```

After the first build:
- The app is built with version `3.0.0.5`
- `release.json` is updated to `"Version": "3.0.0.6"`

After the second build:
- The app is built with version `3.0.0.6`
- `release.json` is updated to `"Version": "3.0.0.7"`

This is useful for **automated CI/CD pipelines** where you want each build to have a unique, monotonically increasing version number without manual intervention.