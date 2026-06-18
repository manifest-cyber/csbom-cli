# csbom-cli

A CLI tool for generating comprehensive **Software Bill of Materials (SBOM)** for C/C++ projects. It analyzes build system files to extract dependencies and enriches them with security identifiers -- CPEs, PURLs, and CVE products -- for vulnerability scanning, **without requiring a build**.

---

## Overview

`csbom` supports multiple C/C++ build systems and package managers:

| Build System   | Files Detected                                    |
| -------------- | ------------------------------------------------- |
| CMake          | `CMakeLists.txt`, `*.cmake`                       |
| Conan v1 & v2  | `conanfile.txt`, `conanfile.py`, `conan.lock`     |
| vcpkg          | `vcpkg.json`, `vcpkg.lock`                        |
| Meson          | `meson.build`                                     |
| pkg-config     | `*.pc`                                            |
| Makefile       | `Makefile`, `*.mk`                                |
| BitBake/Yocto  | `*.bb`, `*.bbappend`                              |

Each extracted component gets:

- **Security identifiers:** CPE 2.3, PURL, CVE products for NVD matching
- **Quality score:** confidence-based scoring (high/medium/low) per field
- **Audit trail:** `strategy` array tracking how each field was discovered

---

## Requirements

- Node.js >= 18.0.0
- (Optional) CMake / Conan installed for enhanced `--install` mode

---

## Usage

```text
Usage: csbom [options] <project-path>

Arguments:
  project-path                 Path to C/C++ project directory

Options:
  -V, --version                output the version number
  --install                    Run build tools (CMake/Conan) for better accuracy
  --verbose                    Show detailed progress logs
  --vendor-dir <dirs>          Comma-separated vendor directories to scan for dependencies
  -o, --output <file>          Output file path (default: "csbom.json")
  -f, --format <format>        Output format: csbom, cyclonedx (default: "csbom")
  --name <name>                Project name for SBOM metadata
  --project-version <version>  Project version for SBOM metadata
  --no-cache                   Disable API response caching
  --clear-cache                Clear all cached API responses and exit
  --clean                      Remove components without valid semantic versions
  -h, --help                   display help for command
```

### Examples

```bash
# Basic scan
csbom /path/to/my-cpp-project

# Enhanced scan using build tools (Conan/CMake)
csbom /path/to/project --install --verbose

# Output as CycloneDX 1.6
csbom /path/to/project -f cyclonedx -o sbom.xml

# Set project metadata and filter unversioned components
csbom /path/to/project --name myapp --project-version 2.1.0 --clean

# Scan specific vendor directories
csbom /path/to/project --vendor-dir third_party,extern

# Clear API cache
csbom --clear-cache
```

---

## Output Format

The default `csbom` JSON format includes:

```json
{
  "metadata": {
    "name": "my-project",
    "version": "1.0.0",
    "timestamp": "...",
    "tool": "csbom"
  },
  "components": [
    {
      "uuid": "...",
      "name": "openssl",
      "version": "3.0.2",
      "originType": "conan",
      "discoveredFrom": ["conanfile.txt"],
      "cpes": ["cpe:2.3:a:openssl:openssl:3.0.2:*:*:*:*:*:*:*"],
      "purls": ["pkg:conan/openssl@3.0.2"],
      "cveProducts": ["openssl:openssl"],
      "licenses": ["Apache-2.0"],
      "quality": "high",
      "strategy": [...]
    }
  ]
}
```

CycloneDX 1.6 output is also available via `-f cyclonedx`.

---

## Pipeline Phases

The tool runs a 10-phase pipeline:

1. **Copy** - Optionally isolate a project copy
2. **Install** - Generate Conan lock files (`--install` only)
3. **Scan** - Discover 14+ file types
4. **Scrap** - Extract from vendor metadata
5. **Parse** - Run 17 specialized parsers
6. **Normalize** - Standardize data
7. **Blend** - Deduplicate components and calculate quality
8. **Enrich** - Add security identifiers (CPE, PURL, CVE)
9. **Clean** - Filter low-quality components
10. **Output** - Write SBOM and clean up

---

## Caching

API responses are cached at `~/.cache/csbom/` with a 7-day TTL. Use `--no-cache` to disable or `--clear-cache` to reset.

---

## Environment Variables

| Variable                     | Purpose                                                   |
| ---------------------------- | --------------------------------------------------------- |
| `GITHUB_TOKEN` or `GH_TOKEN` | Required for BitBake/Yocto enrichment (Poky API access)   |
