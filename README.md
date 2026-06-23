# Cask Exchange Documentation Workspace

This folder is organized as a prompt-ready workspace for creating and refining Cask Exchange business analysis documents.

## Start Here

Use these files first when giving a future prompt:

| File | Purpose |
|---|---|
| `AGENTS.md` | Operating context and rules for future AI assistants |
| `docs/README.md` | Index of source documents and feature specs |
| `docs/project-summary.md` | Current project summary and key decisions |
| `docs/prd-mvp.md` | Product scope, personas, user stories, and MVP requirements |
| `docs/api-reference.md` | Existing API behavior that new API requirements must respect |

## Folder Map

```text
cask-exchange/
├── AGENTS.md
├── README.md
├── docs/
│   ├── README.md
│   ├── prd-mvp.md
│   ├── project-summary.md
│   ├── api-reference.md
│   ├── data-models/
│   ├── wireframes/
│   └── [feature-name]/
├── prompts/
│   └── later-prompt-template.md
└── templates/
    ├── document-1-ui-frontend-specs.md
    └── document-2-api-requirements.md
```

## Standard Feature Output

Each feature should normally live in `docs/[feature-name]/` and include:

| File | Description |
|---|---|
| `ui-frontend-specs.md` | Document 1: UI and frontend behavior |
| `api-requirements.md` | Document 2: backend logic, use cases, business rules, data model, permissions |
| `user-story-summary.md` | Optional summary of user stories and acceptance criteria |
| `user-story-api.md` | Optional API-focused user story detail |

## Prompting Pattern

For best results, future prompts should name:

1. The target feature folder or desired new feature name.
2. Whether the output is UI specs, API requirements, user stories, or a review.
3. Relevant source files to read first.
4. Any decisions that should be preserved.

Use `prompts/later-prompt-template.md` as a copy-ready starting point.
