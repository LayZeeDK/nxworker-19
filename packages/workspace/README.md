# @nxworker/workspace

`@nxworker/workspace` is an Nx plugin that ships the `@nxworker/workspace:move-file` generator for safely moving source files between Nx projects while keeping every import, export, and dependent project in sync.

## Highlights

- Moves files across Nx projects, updating static `import`, dynamic `import()`, and re-export statements automatically
- Understands Nx project graphs: re-wires dependent projects when exported files move and preserves package entrypoints
- Runs with strong input validation (path sanitisation, regex escaping, traversal blocking, optional Unicode opt-in)
- Formats affected files with Prettier unless `--skipFormat` is provided
- Backed by an extensive Jest unit suite and 20+ Verdaccio-powered end-to-end scenarios that exercise OS, architecture, and Node.js edge cases

## Requirements

- Nx 19.8
- Node.js 18, 20, and 22 are officially supported (follows Nx)
- ECMAScript Modules (ESM) only, no CommonJS support

## Platform & Architecture Support

The generator has been validated through automated CI and e2e suites on multiple operating systems and CPU architectures:

- **Linux**: Ubuntu x64/arm64
- **Windows**: Windows Server x64 and Windows 11 arm64
- **macOS**: macOS x64/arm64
- **CPU architectures**: x64/arm64

## Installation and usage

Install the plugin in your Nx workspace and invoke the generator with any Nx-compatible package manager:

```shell
npm install --save-dev @nxworker/workspace

nx generate @nxworker/workspace:move-file <from-file-path> <to-file-path>
```

### Generator options

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `from` | `string` | – | Source file path relative to the workspace root. |
| `to` | `string` | – | Target file path relative to the workspace root. Missing directories are auto-created. |
| `skipExport` | `boolean` | `false` | Skip adding the moved file to the target project's entrypoint (`index.ts`) to ignore the generator's smart defaults. |
| `allowUnicode` | `boolean` | `false` | Permit Unicode characters in the `from`/`to` paths (less safe). |

## Behaviour at a glance

- Determines source/target projects and package import paths automatically via the Nx project graph
- Rewrites imports for:
  - Intra-project relative paths
  - Package alias imports across projects
  - Dynamic `import()` expressions (including chained `.then()` access)
- Updates dependent projects when exported files move, ensuring they resolve the new package alias
- Removes obsolete exports from the source project and adds exports to the target project unless `--skipExport` is set

## Security hardening

- Normalises and sanitises user-supplied paths to block traversal attempts (e.g. `../../..`)
- Rejects regex-special characters that could lead to ReDoS when constructing search expressions
- Provides an explicit `--allowUnicode` escape hatch for workspaces that rely on non-ASCII file names
