# Documentation Index

This folder contains source-of-truth product, API, data model, feature, and wireframe documentation for Cask Exchange.

## Foundation Documents

| File | Use When |
|---|---|
| `project-summary.md` | You need the compact project overview, current decisions, and documentation conventions |
| `prd-mvp.md` | You need product goals, personas, MVP scope, or user journey context |
| `api-reference.md` | You need existing backend/API behavior before writing API requirements |
| `ux-audit-homepage.md` | You need homepage UX audit context |

## Templates

Templates live outside this folder in `../templates/`.

| Template | Use For |
|---|---|
| `../templates/document-1-ui-frontend-specs.md` | UI and frontend behavior specs |
| `../templates/document-2-api-requirements.md` | Backend logic, use cases, business rules, data model, permissions |

## Shared References

| Folder / File | Purpose |
|---|---|
| `data-models/parent-child-structure.md` | Master/Variant cask model and field ownership |
| `data-models/cask-variant-structure.json` | Example cask API response |
| `data-models/checkout-session-structure.json` | Checkout session data example |
| `data-models/schema-change-requests.md` | Proposed schema changes |
| `wireframes/homepage-wireframe-specs.md` | Homepage wireframe notes |
| `wireframes/homepage-midfi.html` | Mid-fidelity homepage wireframe |

## Feature Folders

| Feature Folder | Main Docs |
|---|---|
| `browse-by-category/` | UI specs, API requirements, classification metadata admin |
| `cask-filter/` | UI specs and API requirements for marketplace filtering |
| `curated-list-home/` | UI specs and API requirements for curated home lists |
| `explore-casks-home/` | UI specs and API requirements for Explore Casks home section |
| `market-movers/` | UI specs and API requirements for market mover content |
| `top-distilleries-home/` | UI specs, API requirements, previous API draft, user story docs |

## Recommended Flow For New Feature Specs

1. Read `project-summary.md`, `api-reference.md`, and the relevant data model files.
2. Check whether a similar feature folder already exists.
3. Create or update `docs/[feature-name]/ui-frontend-specs.md`.
4. Create or update `docs/[feature-name]/api-requirements.md`.
5. Add user stories only when requested.
6. Summarize assumptions and open questions at the end.
