---
layout: post
title: "Dev Journal: Shipping Plans and Template Polish"
date: 2026-02-09
categories: [dev-journal]
---

Sunday dev session focused on defining "done" and polishing dotnet templates.

## TIL

### 1. Define "done" before you start
Spent time with a collaborator defining exactly what V1 of a game project means. The scope cage:
- 1 hand-made town (not procedural)
- 5 dungeon levels (not infinite)
- 4 classes (not 12)
- 3 endings (not branching trees)

Ship date: Easter. Slightly uncomfortable = probably correct.

### 2. dotnet templates need explicit parameters
The `fes-module` template was generating broken project references because `--namespace` defaulted to `FesStarter`. Fix: make it required with `"isRequired": true` in template.json.

```json
"namespace": {
  "type": "parameter",
  "isRequired": true,
  "replaces": "FesStarter"
}
```

### 3. Branch protection is worth the friction
Set up GitHub branch protection requiring PR reviews. Immediately caught myself trying to push directly. Good habit even for solo projects.

### 4. Always request Copilot review
GitHub Copilot can review PRs and catch issues. Added it to the workflow - free automated code review, why not?

### 5. Post-actions need explicit file paths
Template post-actions like "add to solution" fail silently without `"projectFiles": ["*.csproj"]` in the args. The error message is unhelpful but the fix is simple.

## Shipped
- fes-starter cleanup (removed redundant AI scaffold skills)
- Fixed module template NuGet references
- Made namespace required across all sub-templates
- Branch protection on fes-starter repo
- Mr. White party game now public

Good Sunday. Time to rest.
