---
argument-hint: "[major|minor|patch|version]"
description: "Prepare a release with version bump and changelog"
model: claude-opus-4-5-20251101
allowed-tools: ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "AskUserQuestion"]
---

**If `$ARGUMENTS` is empty or not provided:**

Analyze commits to suggest the appropriate version bump.

**Usage:** `/release [bump-type|version]`

**Examples:**
- `/release` - Auto-detect bump type from commits
- `/release patch` - Bug fixes only (1.0.0 → 1.0.1)
- `/release minor` - New features (1.0.0 → 1.1.0)
- `/release major` - Breaking changes (1.0.0 → 2.0.0)
- `/release v2.0.0` - Specific version

**Workflow:**
1. Analyze commits since last tag
2. Suggest semantic version bump
3. Generate changelog
4. Update version files
5. Create git tag
6. Draft GitHub release notes

Proceed with auto-detection based on commits.

---

**If `$ARGUMENTS` is provided:**

Prepare a release with the specified version bump or version number.

## Configuration

- **Bump Type**: `$ARGUMENTS` (major, minor, patch, or specific version like v2.0.0)

## Steps

1. **Get Current Version**

   Find current version from:
   ```bash
   # From git tags
   git describe --tags --abbrev=0 2>/dev/null

   # Or from package files
   # package.json: "version": "1.2.3"
   # go.mod: no version (use tags)
   # pyproject.toml: version = "1.2.3"
   # Cargo.toml: version = "1.2.3"
   ```

   If no version found, start at `v0.1.0` or ask user.

2. **Analyze Commits Since Last Release**

   ```bash
   git log $(git describe --tags --abbrev=0)..HEAD --oneline
   ```

   Categorize by conventional commits:
   - `feat:` → suggests minor bump
   - `fix:` → suggests patch bump
   - `BREAKING CHANGE:` → suggests major bump
   - `!` after type (e.g., `feat!:`) → major bump

3. **Determine New Version**

   If bump type provided:
   - `patch`: 1.2.3 → 1.2.4
   - `minor`: 1.2.3 → 1.3.0
   - `patch`: 1.2.3 → 2.0.0

   If auto-detecting:
   - Any `BREAKING CHANGE:` or `!` → major
   - Any `feat:` → minor
   - Only `fix:`, `chore:`, etc. → patch

   Present suggestion to user for confirmation.

4. **Generate Changelog Entry**

   Create entry following Keep a Changelog format:
   ```markdown
   ## [1.3.0] - 2025-11-26

   ### Added
   - New authentication flow (#123)
   - Rate limiting for API endpoints

   ### Fixed
   - Memory leak in connection pool (#456)

   ### Changed
   - Improved error messages

   [1.3.0]: https://github.com/owner/repo/compare/v1.2.3...v1.3.0
   ```

5. **Update Version Files**

   Detect and update version in:
   - `package.json` (Node.js)
   - `pyproject.toml` (Python)
   - `Cargo.toml` (Rust)
   - `version.go` or constants file (Go)
   - `VERSION` file (generic)

   Ask before modifying files.

6. **Update CHANGELOG.md**

   - If exists: Insert new entry at top (below header)
   - If not exists: Ask if user wants to create one
   - Follow existing format if present

7. **Create Release Commit**

   ```bash
   git add -A
   git commit -m "chore(release): v1.3.0

   - Update version to 1.3.0
   - Update CHANGELOG.md"
   ```

8. **Create Annotated Tag**

   ```bash
   git tag -a v1.3.0 -m "Release v1.3.0

   ## Highlights
   - New authentication flow
   - Rate limiting
   - Bug fixes

   See CHANGELOG.md for full details."
   ```

   Always use:
   - Annotated tags (`-a`) not lightweight
   - `v` prefix for consistency
   - Meaningful tag message

9. **Generate GitHub Release Notes**

   Draft release notes:
   ```markdown
   ## What's New in v1.3.0

   ### Highlights
   - 🚀 New authentication flow with JWT support
   - ⚡ Rate limiting to protect API endpoints
   - 🐛 Fixed memory leak affecting long-running processes

   ### Breaking Changes
   None in this release.

   ### Full Changelog
   [v1.2.3...v1.3.0](https://github.com/owner/repo/compare/v1.2.3...v1.3.0)

   ### Contributors
   @contributor1, @contributor2
   ```

10. **Present Summary and Next Steps**

    ```markdown
    ## Release v1.3.0 Ready

    ### Changes Made
    - ✅ Updated package.json to 1.3.0
    - ✅ Updated CHANGELOG.md
    - ✅ Created release commit
    - ✅ Created annotated tag v1.3.0

    ### Next Steps
    1. Review the changes: `git show HEAD`
    2. Push the release: `git push origin main --tags`
    3. Create GitHub release: `gh release create v1.3.0`

    ### Commands to Run
    ```bash
    # Push commit and tag
    git push origin main --tags

    # Create GitHub release (optional)
    gh release create v1.3.0 --generate-notes

    # Or with custom notes
    gh release create v1.3.0 -F release-notes.md
    ```

    Push to remote now? (y/n)
    ```

## Semantic Versioning Rules

| Change Type | Bump | Example |
|-------------|------|---------|
| Breaking API changes | Major | 1.0.0 → 2.0.0 |
| New features (backward compatible) | Minor | 1.0.0 → 1.1.0 |
| Bug fixes (backward compatible) | Patch | 1.0.0 → 1.0.1 |

## Pre-release Versions

Support for pre-releases:
- `v2.0.0-alpha.1`
- `v2.0.0-beta.2`
- `v2.0.0-rc.1`

## Notes

- Always use annotated tags for releases
- Push tags separately: `git push --tags`
- Consider signing tags: `git tag -s`
- Don't push until you've reviewed the changes
- GitHub release can be created from existing tag
