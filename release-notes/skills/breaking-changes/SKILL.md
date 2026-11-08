---
name: breaking-changes
description: Compatibility queries, breaking changes, migration guidance, and TFMs
workflows: https://raw.githubusercontent.com/dotnet/core/refs/heads/main/release-notes/skills/breaking-changes/workflows.json
---

# Breaking Changes Queries

Structured breaking-change queries use manifest → compatibility. Fetch workflows.json for navigation paths with `next_workflow` transitions.

## Data Availability

These workflows require a `compatibility` link to structured JSON with `breaks[]`. If the manifest only provides `compatibility-html`, follow it for documentation, but report structured counts and impact filters as unavailable. Do not construct a compatibility.json URL or interpret missing data as no breaking changes.

## Stop Criteria

| Need | Stop At | Has |
|------|---------|-----|
| Count, categories, impact levels | compatibility.json | `breaks[]` with full metadata |
| Migration steps, code samples | documentation URL | Follow `breaks[].references[type="documentation"]` |

## One Rule

Follow `_links` only. Never construct compatibility.json URLs. Use `impact == "high"` filter for critical review.
