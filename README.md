# command-release

A Claude Code slash command for preparing releases with version bump and changelog.

## Installation

```bash
# Clone to your preferred location
git clone git@github.com:claude-commands/command-release.git <clone-path>/command-release

# Symlink (use full path to cloned repo)
ln -s <clone-path>/command-release/release.md ~/.claude/commands/release.md
```

## Usage

```
/release           # Auto-detect bump type from commits
/release patch     # Bug fixes only (1.0.0 → 1.0.1)
/release minor     # New features (1.0.0 → 1.1.0)
/release major     # Breaking changes (1.0.0 → 2.0.0)
/release v2.0.0    # Specific version
```

## What it does

1. Analyzes commits since last tag
2. Suggests semantic version bump
3. Generates changelog entry
4. Updates version in package files
5. Creates release commit and annotated tag
6. Drafts GitHub release notes

## Output Format

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
```

## Semantic Versioning

| Change Type | Bump | Example |
|-------------|------|---------|
| Breaking changes | Major | 1.0.0 → 2.0.0 |
| New features | Minor | 1.0.0 → 1.1.0 |
| Bug fixes | Patch | 1.0.0 → 1.0.1 |

## Auto-Detection from Commits

| Commit Type | Suggested Bump |
|-------------|---------------|
| `BREAKING CHANGE:` or `feat!:` | Major |
| `feat:` | Minor |
| `fix:`, `chore:`, `docs:` | Patch |

## Supported Version Files

- `package.json` (Node.js)
- `pyproject.toml` (Python)
- `Cargo.toml` (Rust)
- `VERSION` file (generic)
- Git tags (always)

## Requirements

- Git repository with commits
- Claude Code with Sonnet model access

## Updates

```bash
cd <clone-path>/command-release && git pull
```
