# Agent Instructions

This workspace contains BA and product documentation for Cask Exchange, a whisky cask investment marketplace.

## Primary Goal

Help produce consistent, prompt-ready BA documents:

- Document 1: UI & Frontend Specs
- Document 2: Backend Logic & Flow Specs / API Requirements
- User stories and acceptance criteria when requested
- Reviews and refinements of existing BA documentation

## Read Order For Future Work

Before creating or revising feature documentation, read:

1. `docs/project-summary.md`
2. `docs/README.md`
3. `docs/api-reference.md`
4. `templates/document-1-ui-frontend-specs.md` for UI/frontend specs
5. `templates/document-2-api-requirements.md` for API/backend specs
6. Relevant files in `docs/data-models/`
7. The target feature folder under `docs/[feature-name]/`

For broad product context, also read `docs/prd-mvp.md`.

## Documentation Rules

- Keep feature docs in `docs/[feature-name]/`.
- Use kebab-case folder names, for example `market-movers` or `browse-by-category`.
- Follow the templates in `templates/` unless the user explicitly asks for another format.
- Check `docs/api-reference.md` before proposing new backend behavior.
- Do not invent endpoints when the requested document should describe system behavior rather than API design.
- Default permissions to authenticated users unless the feature explicitly requires public access.
- Preserve the parent/child cask model described in `docs/data-models/parent-child-structure.md`.

## Domain Rules To Preserve

- Master Cask groups variants and holds shared attributes.
- Variant holds lifecycle and trade fields such as `readyToSell`, `caskStatus`, `isListed`, `lowestAsk`, and `highestBid`.
- A Master Cask is active when at least one variant has `caskStatus = active` and `isListed = true`.
- Display `classificationLabel` to users, not raw `classification`.
- Region is an object in current API data; expose display values from `region.name` when needed.

## Output Expectations

- Save new feature docs to `docs/[feature-name]/`.
- Summarize key decisions, assumptions, and open questions after changes.
- Avoid moving existing files unless the user specifically asks for a file reorganization.
- Be careful with files already modified in git status; treat them as user work unless the current task requires editing them.
