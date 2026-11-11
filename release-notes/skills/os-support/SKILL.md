---
name: os-support
description: OS packages, distro support, glibc/musl requirements
workflows: https://raw.githubusercontent.com/dotnet/core/refs/heads/main/release-notes/skills/os-support/workflows.json
---

# OS Support Queries

## First: Which File?

| Query | File | Path from manifest |
| ----- | ---- | ------------------ |
| Package dependencies | `os-packages.json` | `os-packages-json` |
| Distro support | `supported-os.json` | `supported-os-json` |
| glibc/musl version | `supported-os.json` | `supported-os-json` |

**Pick ONE file** based on your query—don't fetch both.

## Quick Answers

From `supported-os.json`:

- Is Ubuntu 24.04 supported? → `families[].distributions[?(@.id == 'ubuntu')].supported-versions[]`
- Minimum glibc version? → `libc[?(@.name == 'glibc')]`

From `os-packages.json`:

- What packages for Ubuntu? → `distributions[?(@.name == 'Ubuntu')].releases[].packages[].name`
- How to install them? → The distribution's `install-commands[]`; substitute each package's `name` for `{packageName}`.

Match the distribution's actual `name`, including case. Package entries are objects with `id` and `name`, not strings.

## Stop Criteria

**STOP when you have the JSON file.** Parse it—don't fetch anything else, except when `docker-setup` explicitly chains downloads and OS package queries.

## Workflows

```json
"os-packages": {
  "follow_path": ["kind:llms", "patches.{version}", "manifest", "os-packages-json"],
  "destination_kind": "os-packages",
  "yields": "json"
}
```

```json
"supported-distros": {
  "follow_path": ["kind:llms", "patches.{version}", "manifest", "supported-os-json"],
  "destination_kind": "supported-os",
  "yields": "json"
}
```

```json
"distro-packages": {
  "follow_path": ["kind:os-packages"],
  "filter": "$.distributions[?(@.name == '{distro}')].releases[?(@.release == '{release}')].packages[*].name",
  "yields": "json"
}
```

**Example:** Get Ubuntu 24.04 packages:

```text
filter: $.distributions[?(@.name == 'Ubuntu')].releases[?(@.release == '24.04')].packages[*].name
→ ["ca-certificates", "libc6", "libgcc-s1", ...]
```

All workflows: <https://raw.githubusercontent.com/dotnet/core/refs/heads/main/release-notes/skills/os-support/workflows.json>

## External Schema: supported-os.json

```json
{
  "families": [{
    "name": "Linux",
    "distributions": [{
      "id": "ubuntu",
      "name": "Ubuntu",
      "architectures": ["Arm32", "Arm64", "x64"],
      "supported-versions": ["24.04", "22.04"]
    }]
  }],
  "libc": [
    { "name": "glibc", "version": "2.27", "architectures": ["Arm64", "x64"] },
    { "name": "musl", "version": "1.2.3", "architectures": ["Arm64", "x64"] }
  ]
}
```

## External Schema: os-packages.json

```json
{
  "distributions": [{
    "name": "Ubuntu",
    "install-commands": [
      { "run-under-sudo": true, "command-root": "apt-get", "command-parts": ["update"] },
      { "run-under-sudo": true, "command-root": "apt-get", "command-parts": ["install", "-y", "{packageName}"] }
    ],
    "releases": [{
      "release": "24.04",
      "packages": [
        { "id": "libc", "name": "libc6" },
        { "id": "icu", "name": "libicu74" }
      ]
    }]
  }]
}
```

The examples show only part of each dataset. In `supported-os.json`, match distribution `id`; in `os-packages.json`, match the actual distribution `name` (for example, `Ubuntu` or `Alpine`).

## Common Mistakes

| Mistake | Why It's Wrong |
| ------- | -------------- |
| Looking for `.md` versions | Don't exist—only `.json` |
| Fetching both JSON files | Pick ONE based on query |
| Constructing URLs | Follow `_links` from manifest |

## Tips

- Package names vary by distro version (e.g., `libicu74` vs `libicu72`)
- Alpine uses musl, not glibc
- Newer .NET may drop older distro support—check specific version
