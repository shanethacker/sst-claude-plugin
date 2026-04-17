# Research Guide for Dependency Analysis

## Where to find information

### Changelogs and release notes
- **PyPI**: `https://pypi.org/project/<package>/` — check "Release history" tab
- **npm**: `https://www.npmjs.com/package/<package>?activeTab=versions` — check versions and links to changelog
- **crates.io**: `https://crates.io/crates/<package>/versions` — Rust packages
- **GitHub releases**: Most packages link from their registry page to GitHub; check the Releases page
- **CHANGELOG.md**: Many repos keep a changelog in their root; check the repo directly
- **Context7 MCP**: For major packages, use the Context7 MCP server to get authoritative, up-to-date documentation

### Security advisories
- **GitHub Advisory Database**: `gh api /advisories?ecosystem=<pip|npm|cargo>&package=<name>` or search via WebSearch
- **PyPI**: Security advisories sometimes appear on the package page
- **npm audit**: For Node packages, advisories appear in the npm registry
- **CVE databases**: Search `<package> CVE` via WebSearch for recent vulnerabilities
- **Dependabot PR body**: Often includes advisory links when the update is security-motivated

### Compatibility information
- **PyPI classifiers**: Check "Programming Language :: Python :: <version>" and framework classifiers
- **npm engines field**: `package.json` `engines` field specifies supported Node versions
- **Package docs**: Look for a compatibility matrix or "supported versions" page
- **CI configs**: Check the package's GitHub repo CI for tested runtime/framework versions
- **Tox/nox configs**: Python packages often list tested version combinations in `tox.ini` or `noxfile.py`

### Cross-dependency relationships
- **PyPI "Requires"**: Shows what a package depends on (visible on PyPI project page)
- **npm "dependencies"**: Visible in `package.json` and on the npm registry page
- Read the project's dependency manifest constraints carefully — look for version ranges, not just pins

## Search strategies

When using WebSearch for a package update:
1. `"<package> <new_version> changelog"` — direct hit on release notes
2. `"<package> <new_version> breaking changes"` — surface known issues
3. `"<package> <new_version> <framework> <version>"` — compatibility reports (e.g., Django 5.2, React 19)
4. `"<package> <new_version> <runtime> <version>"` — runtime compatibility (e.g., Python 3.13, Node 22)
5. `"<package> CVE 2025 2026"` — recent security issues

## Project-specific context to gather before starting

Before beginning research, examine the repo to determine:
- **Language/runtime version** (e.g., Python 3.13, Node 22, Rust 1.80)
- **Framework version** (e.g., Django 5.2, Next.js 15, Axum 0.7)
- **Dependency ecosystems** — check `dependabot.yml` for configured directories and package ecosystems
- **Pinning strategy** — exact pins (`==`, `=`), ranges (`>=x,<y`), or semver (`^`, `~`)
- **Lockfile format** — identify which lockfile is in use and how it's regenerated (e.g., `uv lock`, `npm install`, `cargo update`)
- **CI pipeline** — what checks run on PRs (tests, lint, type checking, build)
