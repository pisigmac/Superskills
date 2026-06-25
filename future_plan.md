# Future Plan for Superskills

This document captures the current gaps, planned fixes, and enhancements for the Superskills project.

## Current Issues

1. **Remaining placeholders**
   - `README.md` still contains `[RELEASE_POST_URL]` and `[CLAUDE_MARKETPLACE_URL]`.
   - `CODE_OF_CONDUCT.md` still contains `[AUTHOR_EMAIL]`.
   - These placeholders should be replaced with real values or removed before public launch.

2. **Missing plugin manifests**
   - No `.cursor-plugin/plugin.json` manifest for Cursor.
   - No marketplace catalog for GitHub Copilot CLI.
   - Cursor and Copilot CLI installs remain marked "in progress" in the README.

3. **Referenced files that do not exist**
   - `AGENTS.md` references `.github/PULL_REQUEST_TEMPLATE.md`, which is not present.
   - `package.json` references `.opencode/plugins/superskills.js`, which is not present.

4. **Icon file size and bit depth**
   - `assets/app-icon.png` is 16-bit RGBA and ~860 KB.
   - Should be re-exported as 8-bit sRGB to reduce size and improve compatibility.

5. **Untested Feature Factory skills**
   - The 10 new Feature Factory skills were adapted from prompts but have not been validated in live sessions.
   - No dedicated tests exist for the new pipeline.

6. **Internal process artifacts**
   - `docs/superskills/specs/2026-06-24-feature-factory-integration-design.md` is a working design doc.
   - It may confuse end users browsing documentation.

7. **Mixed installation messaging**
   - The README shows marketplace commands as "in progress" next to direct repo install commands.
   - A clearer top-level note about which installs work today would help users.

## Deployment Rules

- All code changes must be pushed to the `dev` branch first.
- If the `dev` branch does not exist, create it from `main` before pushing.
- Pushing directly to `main` is not allowed without explicit user confirmation.
- Before merging `dev` into `main`, the user must review and approve the changes.

## Planned Enhancements

### Documentation
- [ ] Add `CONTRIBUTING.md` summarizing the rules in `AGENTS.md`.
- [ ] Add `CHANGELOG.md` starting from the Superskills rebrand.
- [ ] Add a getting-started example showing a full Feature Factory session.
- [ ] Add `SECURITY.md` with a vulnerability reporting process.

### Packaging
- [ ] Create `.cursor-plugin/plugin.json` for Cursor.
- [ ] Create a marketplace catalog JSON for GitHub Copilot CLI.
- [ ] Re-compress `assets/app-icon.png` to 8-bit sRGB.
- [ ] Add `.gitattributes` or harness-specific ignore files if needed.

### Quality
- [ ] Run existing Claude Code and OpenCode skill tests.
- [ ] Add a YAML frontmatter validator for all skill files.
- [ ] Add a GitHub Action that validates skill frontmatter on pull requests.
- [ ] Pressure-test the Feature Factory pipeline in a sample project.

### Community
- [ ] Enable GitHub Discussions and issue templates.
- [ ] Publish a GitHub release so the README release-announcements link is meaningful.
- [ ] Add a "Used by" or testimonials section once early adopters exist.

## Acceptance Criteria

This plan is complete when:
- All placeholders are resolved or removed.
- Every harness listed in the README has a working install path or a clear "coming soon" note.
- The Feature Factory skills have been exercised in at least one end-to-end session.
- A CI check validates skill frontmatter on every PR to `dev`.
