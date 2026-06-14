# Release SOP

This document defines the standard release procedure for the `openclaw-skills` repository.

## When to release

Create a release when one of the following is true:

- A new skill is added
- An existing skill gains meaningful new behavior
- A skill workflow is changed in a way worth tracking
- A stable batch of edits is ready to become the new canonical version

## Version decision rules

- **patch**: wording cleanup, typo fixes, small clarifications, no meaningful behavior change
- **minor**: new skill, notable expansion, new branch in workflow, stronger guidance
- **major**: breaking change, removed assumptions, incompatible workflow update

## Release steps

1. Review local changes
   ```bash
   git status
   git diff
   ```

2. Update affected skill metadata
   - bump `version`
   - refresh `last_updated`

3. Update repository docs if needed
   - `README.md`
   - `CHANGELOG.md`
   - `RELEASE.md`

4. Commit the release
   ```bash
   git add .
   git commit -m "chore: prepare vX.Y.Z"
   ```

5. Create the tag
   ```bash
   git tag -a vX.Y.Z -m "vX.Y.Z"
   ```

6. Push code and tag
   ```bash
   git push origin main
   git push origin vX.Y.Z
   ```

7. Verify remote state
   ```bash
   git ls-remote --heads --tags origin
   ```

## Minimal release standard

A release is considered valid only if:

- GitHub `main` contains the release commit
- the tag exists on GitHub
- `CHANGELOG.md` reflects the release
- affected `SKILL.md` files show the intended version

## Recommended discipline

- Do not leave long-lived unpublished local changes
- Prefer small, readable releases over large bundled updates
- Tag every stable public milestone
- Treat GitHub as the only canonical reference when questions arise
