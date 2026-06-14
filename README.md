# openclaw-skills

Custom OpenClaw skills maintained in GitHub as the canonical source of truth.

## Included skills

| Skill | Purpose |
|---|---|
| `ingest` | Process new notes from `Products/Notes/` and route them into the right knowledge directories after approval. |
| `lint` | Audit the `Products/` knowledge base for orphan pages, stale content, missing links, and note backlog. |
| `psych` | Provide emotional processing and psychological support when the user needs reflection rather than action-pushing. |
| `query` | Answer questions by searching and synthesizing the user's `Products/` knowledge base first. |
| `save` | Save the current conversation as a structured Markdown note in `Products/Notes/Conversation/`. |
| `scan-x-accounts` | Monitor selected X accounts and extract A/B category topic signals from recent high-value posts. |
| `scan-x-primitives` | Mine X for A-category tool tutorial topics using the five action primitives framework. |
| `scan-x-trending` | Scan X for B-category hot news and product-analysis topic opportunities. |

## Repository structure

```text
.
├── CHANGELOG.md
├── README.md
├── RELEASE.md
├── ingest/
├── lint/
├── psych/
├── query/
├── save/
├── scan-x-accounts/
├── scan-x-primitives/
└── scan-x-trending/
```

## Versioning policy

This repository uses a light semantic versioning approach.

### Repository as source of truth

- GitHub `main` is the canonical stable version.
- Local edits are only working changes until they are committed and pushed.
- Every stable release should have a Git tag.

### Skill-level metadata

Each `SKILL.md` should keep these front matter fields up to date:

```yaml
version: 0.1.0
last_updated: 2026-06-14
```

### Version bump rules

- **patch**: wording cleanup, small fixes, non-behavioral clarifications
  - example: `0.1.0 -> 0.1.1`
- **minor**: new skill, meaningful capability expansion, new workflow branch
  - example: `0.1.0 -> 0.2.0`
- **major**: breaking behavior change, incompatible workflow change, removed prior assumptions
  - example: `0.1.0 -> 1.0.0`

## Release checklist

Before every release:

- [ ] All intended changes are committed
- [ ] `git status` is clean
- [ ] Updated `README.md` if conventions changed
- [ ] Updated affected `SKILL.md` `version` fields
- [ ] Updated affected `SKILL.md` `last_updated` fields
- [ ] Updated `CHANGELOG.md`
- [ ] Created a release tag
- [ ] Pushed `main`
- [ ] Pushed tags

## Recommended release flow

```bash
git status
git add .
git commit -m "chore: prepare vX.Y.Z"
git tag -a vX.Y.Z -m "vX.Y.Z"
git push origin main
git push origin vX.Y.Z
```

## Changelog

See `CHANGELOG.md` for release history.
