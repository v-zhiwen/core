---
name: version-eol
description: EOL versions, support lifecycle, version history, and release types
workflows: https://raw.githubusercontent.com/dotnet/core/refs/heads/main/release-notes/skills/version-eol/workflows.json
---

# Version and EOL Queries

Supported versions are embedded in llms.json. EOL versions require root → major index. Fetch workflows.json for navigation paths with `next_workflow` transitions.

## Stop Criteria

| Need | Stop At | Has |
|------|---------|-----|
| Supported version info (patch, LTS/STS, support phase) | llms.json | `_embedded.patches["X.0"]` |
| EOL date for any version | major index | `eol_date`, `supported` |
| Last security patch for EOL version | major index | `latest_security_patch`, `latest_security_patch_date` |
| CVE details for that patch | security month, then cve.json if linked | `_embedded.disclosures[]`, when available |

If the security month has no disclosures or `cve-json` link, report CVE details unavailable. This does not mean no CVEs were fixed.

## One Rule

Check `supported_releases` first. If version is listed, use `_embedded.patches`. Otherwise navigate via `root`.

In workflow paths, `releases.{version}` selects the matching `version` in `root._embedded.releases[]`. Follow that entry's `self` link to the major index; the root has no link relation named after each version.
