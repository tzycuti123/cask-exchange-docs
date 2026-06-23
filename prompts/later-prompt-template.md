# Later Prompt Template

Use this template when asking an AI assistant to continue work in this folder.

```text
You are working in the Cask Exchange documentation workspace.

First read:
- AGENTS.md
- docs/README.md
- docs/project-summary.md
- docs/api-reference.md
- templates/document-1-ui-frontend-specs.md
- templates/document-2-api-requirements.md
- [add any feature-specific files here]

Task:
[Describe the feature or document you want created/reviewed/refined.]

Target output:
- [UI specs / API requirements / user stories / review / rewrite]
- Save output to: docs/[feature-name]/[file-name].md

Important context to preserve:
- [Decision 1]
- [Decision 2]
- [Any API constraints, entity rules, or UX assumptions]

Before finishing:
- Check consistency with docs/api-reference.md
- Check the parent/child cask model in docs/data-models/
- Summarize assumptions, key decisions, and open questions
```

## Example

```text
Create UI and API specs for a "Watchlist" feature.

Read AGENTS.md, docs/README.md, docs/project-summary.md, docs/api-reference.md,
docs/data-models/parent-child-structure.md, and both templates.

Save:
- docs/watchlist/ui-frontend-specs.md
- docs/watchlist/api-requirements.md

The feature lets authenticated users save Master Casks to a personal watchlist.
Preserve the rule that Master active state is derived from variants.
```
