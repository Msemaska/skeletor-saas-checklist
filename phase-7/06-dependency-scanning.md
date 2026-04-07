# Dependency Scanning

---

## What It Is

Automated detection of known security vulnerabilities in your npm dependencies.

## Tool

Use GitHub Dependabot. It's built into GitHub, requires no additional setup beyond enabling it, and creates pull requests to update vulnerable dependencies automatically.

## Configuration

**Design decisions:**
- Enable Dependabot security updates — automatic PRs for dependencies with known vulnerabilities
- Enable Dependabot version updates — automatic PRs for outdated dependencies (weekly schedule)
- Review and merge Dependabot PRs within 48 hours for security updates. Version updates can be batched weekly.
- Pin all dependencies to exact versions in `package-lock.json` (npm does this by default) — don't use `^` or `~` ranges in `package.json` for production dependencies

## npm audit

In addition to Dependabot, run `npm audit` as part of the CI pipeline.

**Design decisions:**
- Run `npm audit` in the GitHub Actions CI workflow on every push
- Fail the build on critical and high severity vulnerabilities — these must be resolved before merging
- Moderate and low severity vulnerabilities are logged as warnings but don't block the build
- Run `npm audit fix` to auto-resolve vulnerabilities when possible. If `npm audit fix` requires a major version bump, review the changelog before upgrading.

## Dependency Hygiene

Beyond automated scanning, maintain healthy dependencies:

- **Audit before adding** — before installing a new dependency, check: maintenance status (last published, open issues, contributor count), download count, bundle size impact, and whether a built-in or existing dependency already handles the use case
- **Remove unused dependencies** — periodically check for packages that are installed but no longer imported. Tools like `depcheck` detect these.
- **Prefer well-maintained packages** — a dependency with one maintainer and no updates in 2 years is a risk, regardless of whether it has known vulnerabilities today

## What to Test

- Dependabot is enabled and creating PRs for vulnerable dependencies
- `npm audit` runs in CI and fails on critical/high severity issues
- No critical or high severity vulnerabilities in the current dependency tree
- All dependencies are pinned to exact versions in package-lock.json

## Edge Cases

- **Vulnerability has no fix available** — the maintainer hasn't patched the vulnerability yet. Assess whether the vulnerability is exploitable in your usage of the package and document the decision.
- **Dependabot PR breaks the build** — the updated dependency has breaking changes. Review the changelog, update your code, and merge. Don't ignore the PR — the vulnerability is still there.
- **Transitive dependency vulnerability** — a vulnerability is in a dependency of a dependency. `npm audit fix` may not resolve it if the parent dependency hasn't updated. You may need to override the transitive dependency version using npm overrides.
- **False positive** — the vulnerability exists in the package but isn't exploitable in your usage (e.g., a vulnerability in a CLI feature of a package you only use as a library). Document the assessment and add it to the audit exceptions.

## Implementation Checklist

- [ ] Dependabot security updates enabled in the repository
- [ ] Dependabot version updates enabled (weekly schedule)
- [ ] `npm audit` runs in CI pipeline on every push
- [ ] CI fails on critical and high severity vulnerabilities
- [ ] All dependencies pinned to exact versions
- [ ] Security update PRs reviewed and merged within 48 hours
- [ ] No critical or high severity vulnerabilities at launch

## Common Mistakes

- Dependabot not enabled (vulnerable dependencies sit undetected for months)
- Dependabot PRs ignored (PRs pile up, vulnerabilities remain unpatched)
- No `npm audit` in CI (vulnerabilities introduced by new dependencies aren't caught until Dependabot's next scan)
- Using `^` version ranges for production dependencies (an automatic minor update could introduce a vulnerability or breaking change)
- Adding dependencies without checking maintenance status (abandoned package becomes a liability)
- Ignoring transitive dependency vulnerabilities (the vulnerability is real even if it's not in your direct dependency)
