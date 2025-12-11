# setup-pwsh

[![GitHub Action](https://img.shields.io/badge/GitHub-Action-blue?logo=github)](https://github.com/mchave3/setup-pwsh)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A GitHub Action to download and install any version of **PowerShell Core** on GitHub Actions runners.

## Features

- ✅ **Cross-platform**: Works on Windows, macOS, and Linux runners
- ✅ **Flexible versioning**: Install latest stable, preview, or specific versions
- ✅ **Multi-architecture**: Supports x64, x86, ARM64, and ARM32
- ✅ **Caching**: Uses runner tool cache for faster subsequent runs
- ✅ **Automatic detection**: Auto-detects OS and architecture

## Usage

### Basic Usage - Latest Version

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup PowerShell
    uses: mchave3/setup-pwsh@v1
    with:
      github-token: ${{ github.token }}

  - name: Run PowerShell script
    shell: pwsh
    run: |
      $PSVersionTable
```

### Install LTS/Stable Version (7.4.x)

```yaml
steps:
  - name: Setup PowerShell LTS
    uses: mchave3/setup-pwsh@v1
    with:
      version: 'stable'
      github-token: ${{ github.token }}
```

### Install Specific Version

```yaml
steps:
  - name: Setup PowerShell 7.4.6
    uses: mchave3/setup-pwsh@v1
    with:
      version: '7.4.6'
      github-token: ${{ github.token }}
```

### Install Latest Preview

```yaml
steps:
  - name: Setup PowerShell Preview
    uses: mchave3/setup-pwsh@v1
    with:
      version: 'preview'
      github-token: ${{ github.token }}
```

### Specify Architecture

```yaml
steps:
  - name: Setup PowerShell ARM64
    uses: mchave3/setup-pwsh@v1
    with:
      version: 'stable'
      architecture: 'arm64'
      github-token: ${{ github.token }}
```

### Matrix Testing

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        pwsh-version: ['7.2.0', '7.4.0', 'stable', 'preview']

    runs-on: ${{ matrix.os }}

    steps:
      - uses: actions/checkout@v4

      - name: Setup PowerShell
        uses: mchave3/setup-pwsh@v1
        with:
          version: ${{ matrix.pwsh-version }}

      - name: Test PowerShell
        shell: pwsh
        run: |
          Write-Host "PowerShell Version: $($PSVersionTable.PSVersion)"
          Write-Host "OS: $($PSVersionTable.OS)"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | PowerShell version to install | No | `latest` |
| `architecture` | Target architecture | No | `auto` |
| `github-token` | GitHub token for API authentication (recommended) | No | `""` |

### Version Options

| Value | Description |
|-------|-------------|
| `latest` | Latest release (currently 7.5.x) - from /releases/latest |
| `stable` or `lts` | Latest LTS release (currently 7.4.x - supported until Nov 2026) |
| `preview` | Latest preview/RC release |
| `7.5.4` | Specific version (e.g., 7.5.4, 7.4.6) |

### PowerShell Support Lifecycle

| Version | Type | End of Support |
|---------|------|----------------|
| 7.5.x | Current | May 2026 |
| 7.4.x | **LTS** | **Nov 2026** |
| 7.2.x | LTS (EOL) | ~~Nov 2024~~ |

> ⚠️ **Note**: PowerShell 7.2.x reached end-of-support in November 2024. Use `stable` or `lts` to get the current LTS version (7.4.x).

### Architecture Options

| Value | Description | Windows | macOS | Linux |
|-------|-------------|---------|-------|-------|
| `auto` | Auto-detect (default) | ✅ | ✅ | ✅ |
| `x64` | 64-bit Intel/AMD | ✅ | ✅ | ✅ |
| `x86` | 32-bit | ✅ | ❌ | ❌ |
| `arm64` | ARM 64-bit | ✅ | ✅ | ✅ |
| `arm32` | ARM 32-bit | ❌ | ❌ | ✅ |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The installed PowerShell version |
| `path` | The installation path |

### Using Outputs

```yaml
steps:
  - name: Setup PowerShell
    id: setup-pwsh
    uses: mchave3/setup-pwsh@v1
    with:
      version: 'stable'

  - name: Display installed version
    run: |
      echo "Installed version: ${{ steps.setup-pwsh.outputs.version }}"
      echo "Installation path: ${{ steps.setup-pwsh.outputs.path }}"
```

## Supported Runners

| Runner | Status |
|--------|--------|
| `ubuntu-latest` | ✅ Supported |
| `ubuntu-22.04` | ✅ Supported |
| `ubuntu-20.04` | ✅ Supported |
| `windows-latest` | ✅ Supported |
| `windows-2022` | ✅ Supported |
| `windows-2019` | ✅ Supported |
| `macos-latest` | ✅ Supported |
| `macos-14` (ARM64) | ✅ Supported |
| `macos-13` | ✅ Supported |
| `self-hosted` | ✅ Supported |

## Examples

### CI/CD Pipeline

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PowerShell
        uses: mchave3/setup-pwsh@v1
        with:
          version: 'stable'

      - name: Run tests
        shell: pwsh
        run: |
          ./scripts/Run-Tests.ps1

      - name: Build
        shell: pwsh
        run: |
          ./scripts/Build.ps1
```

### Multi-version Testing

```yaml
name: Test Multiple PowerShell Versions

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        pwsh: ['7.2.0', '7.3.0', '7.4.0', '7.5.0']

    steps:
      - uses: actions/checkout@v4

      - name: Setup PowerShell ${{ matrix.pwsh }}
        uses: mchave3/setup-pwsh@v1
        with:
          version: ${{ matrix.pwsh }}

      - name: Run tests
        shell: pwsh
        run: |
          Invoke-Pester -EnableExit
```

## How It Works

1. **Detects** the runner's operating system and architecture
2. **Fetches** release information from [PowerShell GitHub releases](https://github.com/PowerShell/PowerShell/releases)
3. **Downloads** the appropriate package (ZIP for Windows, tar.gz for Unix)
4. **Extracts** to the runner tool cache
5. **Adds** the installation path to `$PATH`

## Troubleshooting

### Rate Limiting

To avoid GitHub API rate limiting, **always pass the `github-token`** input:

```yaml
steps:
  - name: Setup PowerShell
    uses: mchave3/setup-pwsh@v1
    with:
      github-token: ${{ github.token }}
```

> ⚠️ **Important**: Without a token, anonymous API requests are limited to 60/hour. With `github.token`, you get 5,000/hour.

### Version Not Found

Ensure the version exists on [PowerShell releases](https://github.com/PowerShell/PowerShell/releases). Use exact version numbers like `7.4.0`, not `7.4`.

### Architecture Not Supported

Not all architectures are available for all platforms:
- `x86` is only available on Windows
- `arm32` is only available on Linux

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.