# CLI

---

## What It Is

A Node.js command-line tool that gives developers a first-class terminal interface to your public API. The CLI is a thin client — every command translates into an authenticated HTTP request to the versioned REST API (phase-4-public-api/01-api-foundation.md). It shares the same API keys, rate limits, and scoping rules as any other integration. No database access, no internal routes, no special privilege.

Developers reach for the CLI during onboarding, in CI/CD pipelines, and for one-off operations that would take several API calls to script manually. A well-designed CLI reduces support burden and accelerates integration. It is not a power-user afterthought — it is part of the public API surface.

## Technology

Node.js with Commander.js. Commander.js is the standard for structured argument parsing, subcommand organisation, help generation, and option validation. It is well-maintained, widely adopted, and has no meaningful competition for this use case. Publish the CLI as an npm package so developers install it with `npm install -g your-cli-name` or run it without installing using `npx your-cli-name`.

The package name should match the product name or its common short form. Keep it lowercase, hyphenated, and collision-free on the npm registry. Reserve the name early.

## Authentication

Use browser-based device login as the primary auth flow. When a developer runs `login`, the CLI opens a browser tab to your app's OAuth authorisation page, starts a temporary local HTTP listener on a random port, and waits for the callback. The user approves the request in the browser. The app redirects back to `localhost` with a short-lived authorisation code. The CLI exchanges the code for a scoped API token server-side and stores it locally.

If no browser is available — headless servers, SSH sessions, CI environments — fall back automatically to the Device Authorization Grant (device code flow). The CLI prints a short URL and a user code. The developer visits the URL in any browser, enters the code, and approves. The CLI polls the token endpoint until approval is confirmed.

Both flows produce the same result: a scoped API token stored in the OS keychain. Use the OS native keychain — Windows Credential Manager, macOS Keychain, Linux Secret Service via libsecret — not a plain file in the user's home directory. A plain file ends up in backups, git repositories, and shared archives. The `keytar` package (or its maintained successor) abstracts all three platforms behind a single API.

The stored token must be scoped to CLI operations only. It is not a full workspace API key and should not be reusable as one. Create a dedicated token type in your API key management system (phase-4-public-api/01-api-foundation.md) with a `cli_` prefix so it is visually distinct and can be revoked independently from developer-issued API keys.

Provide a `logout` command that deletes the token from the keychain and revokes it server-side via the API. Revocation must happen server-side — deleting from the keychain alone leaves a live credential on the server.

## Commands

Organise commands into noun-first subcommands that mirror your API resource structure. A command that lists workspaces is `workspaces list`, not `list-workspaces`. This keeps the command tree predictable and auto-complete friendly.

Every command that reads or modifies a resource must produce output in one of two modes, controlled by a global `--json` flag:

- **Human-readable (default):** Formatted, coloured output designed for a terminal. Tables, labels, and status indicators are fine. Prioritise clarity over density.
- **Machine-readable (`--json`):** Plain JSON to stdout, no colour codes, no decorative characters. The shape matches the API response body exactly — no renaming, no reshaping. Errors in JSON mode write to stderr so stdout remains parseable.

Errors always write to stderr regardless of output mode. This means a script can pipe stdout to `jq` without worrying that error text will corrupt the JSON stream.

Never print secrets, tokens, or API keys to stdout or stderr. When displaying a newly created token, print it once and make clear it will not be shown again — matching the behaviour of the web dashboard.

## Configuration

The CLI reads configuration from two places, in order of precedence: environment variables, then the OS keychain / config file. Environment variables take precedence so CI pipelines can override everything without touching stored config.

The environment variable for the API token is `YOUR_PRODUCT_API_TOKEN`. When this variable is set, the CLI skips keychain lookup entirely. This is the recommended pattern for CI/CD — generate a token in the dashboard, store it as a CI secret, and pass it as an environment variable.

Non-secret config (base URL overrides, default workspace, output format preference) lives in a config file at the platform-standard location: `~/.config/your-cli-name/config.json` on Linux and macOS, `%APPDATA%\your-cli-name\config.json` on Windows. Use the `env-paths` package to resolve the correct path per platform rather than hardcoding Unix paths.

## Versioning and Distribution

The CLI version in `package.json` follows semver. Bump the patch version for bug fixes, minor for new commands or flags, major for breaking changes to command names, output shapes, or config format.

On every startup, the CLI checks whether a newer version is available on the npm registry. Do this asynchronously and non-blockingly — the command runs immediately, and the update notice prints after the output. Never delay a command to perform an update check. Use the `update-notifier` package for this. The check result is cached for 24 hours so it does not hit the registry on every invocation.

When the installed CLI version is older than the minimum version required by the API, the API returns a `426 Upgrade Required` response. The CLI must handle this explicitly: detect the 426 status, print a clear upgrade instruction, and exit with a non-zero code. Do not let the command silently fail or print a confusing error about an unexpected response shape.

## CI/CD Usage

The CLI must work without any interactive input in a CI environment. Every prompt that appears in interactive mode must have a corresponding flag that suppresses it. A `--yes` or `--force` flag confirms destructive actions non-interactively. Prompts that cannot be suppressed are a CI-breaking bug.

The CLI exits with code 0 on success, non-zero on any error. Scripts test `$?` — an exit code of 0 that accompanies an error message on stderr is a broken contract.

## Security Considerations

**Token storage:** Tokens must live in the OS keychain, never in plaintext files. Keychain storage encrypts at rest and restricts access to the logged-in user's session. Plaintext config files in the home directory are routinely captured in backups, synced via cloud storage, and accidentally committed to version control.

**Token scope:** CLI tokens are scoped exclusively to CLI operations. They cannot be used in place of developer API keys and carry only the permissions required for CLI commands. Over-privileged CLI tokens become a high-value target if exposed.

**Token revocation:** Logout must revoke the token server-side. Keychain deletion alone is insufficient — the credential remains valid on the server until it expires or is explicitly revoked. The revocation endpoint must be callable without authentication (the token being revoked authenticates the request).

**Secret suppression:** The CLI must never print API keys, tokens, or other credentials to stdout or stderr during normal operation. Log levels must not expose headers containing `Authorization` values. Debug modes that might log request headers must be explicitly opt-in and must redact authorization headers before printing.

**Environment variable leakage:** When `YOUR_PRODUCT_API_TOKEN` is set via environment variable, the CLI must not echo it in output, error messages, or debug traces.

**Minimum version enforcement:** Outdated CLI versions may use deprecated or removed API endpoints. The `426 Upgrade Required` response and the update notifier together ensure users are nudged to stay current. Critical security fixes should increment the minimum required version in the API to force upgrades.

## Deliverables / Implementation Checklist

**Package setup**
- [ ] npm package created with correct name, reserved on the registry
- [ ] Commander.js installed and command tree scaffolded
- [ ] `bin` entry in `package.json` pointing to the CLI entrypoint
- [ ] Semver versioning policy documented in the repo

**Authentication**
- [ ] Browser-based login flow implemented (loopback localhost callback)
- [ ] Device code flow fallback implemented for headless environments
- [ ] OS keychain storage implemented via `keytar` (or equivalent) for all three platforms
- [ ] CLI token type created in API key system with `cli_` prefix
- [ ] `login` command opens browser and stores token on success
- [ ] `logout` command revokes token server-side and deletes from keychain
- [ ] `YOUR_PRODUCT_API_TOKEN` environment variable bypasses keychain for CI use

**Commands**
- [ ] Noun-first subcommand structure implemented (`resource action`)
- [ ] Global `--json` flag implemented on all commands
- [ ] Human-readable output uses formatted, coloured display
- [ ] JSON output writes clean JSON to stdout, errors to stderr only
- [ ] No secrets or tokens printed to stdout or stderr
- [ ] All destructive commands support `--yes` / `--force` for non-interactive use

**Configuration**
- [ ] Config file stored at platform-standard path using `env-paths`
- [ ] Environment variable takes precedence over stored config

**Distribution and updates**
- [ ] `update-notifier` implemented with 24-hour cache
- [ ] Update notice prints after command output, never before
- [ ] `426 Upgrade Required` response handled with explicit upgrade instruction
- [ ] CLI published to npm registry via CI using trusted publishing (OIDC)
- [ ] `npx` usage documented as zero-install alternative

**Exit codes**
- [ ] All commands exit 0 on success, non-zero on any error
- [ ] Errors written to stderr in both output modes

## What to Test

- `login` completes the browser flow and stores a token in the keychain
- `login` falls back to device code flow when no browser is available
- `logout` revokes the token server-side and removes it from the keychain
- `YOUR_PRODUCT_API_TOKEN` environment variable is used when set, skipping keychain
- Every command produces valid JSON when run with `--json`
- JSON output contains no ANSI colour codes
- Error messages in `--json` mode go to stderr, not stdout
- Stdout output when piped through `jq` parses cleanly without errors
- Commands exit with non-zero code on API errors
- Commands exit with non-zero code on network failures
- `426` response triggers the upgrade instruction and a non-zero exit
- `--yes` flag suppresses all interactive prompts
- Secrets and tokens are not printed in stdout or stderr under any flag
- Update check runs asynchronously and does not delay command output
- Installed version older than API minimum triggers correct error
- Token cannot be used as a developer API key against the API
- Login with an already-authenticated session prompts to re-login or overwrites cleanly

## Edge Cases

- **Token expires mid-command** — a long-running command (polling, bulk operation) starts with a valid token that expires partway through; detect the 401 mid-flight, prompt the user to run `login` again, and exit with a non-zero code rather than silently continuing with partial results.
- **CLI version older than API minimum version** — the API returns 426; the CLI must catch this status code explicitly, print a human-readable upgrade message, and exit non-zero rather than crashing with an unhandled response shape error.
- **Output piped to a script** — the developer runs `your-cli resources list --json | jq .data`; any text written to stdout that is not valid JSON (update notices, prompts, debug output) breaks the pipe; all non-JSON output must go to stderr.
- **Rate limit hit inside a scripted loop** — the developer runs the CLI in a bash loop and hits the 429 rate limit from phase-4-public-api/01-api-foundation.md; the CLI must print the `Retry-After` value to stderr and exit non-zero so the script can back off rather than retrying immediately and burning the remaining window.
- **Secrets accidentally printed in verbose or debug mode** — a developer adds `--debug` to diagnose a problem and the CLI prints request headers including `Authorization: Bearer cli_abc123`; implement a header redaction layer that replaces the token value with `[REDACTED]` in all log output regardless of verbosity level.
- **Keychain unavailable on Linux** — the `libsecret` backend is not installed or the session has no keyring daemon; detect the failure gracefully and fall back to writing the token to a file in the config directory with `0600` permissions, with a visible warning that keychain storage is unavailable.
- **Multiple accounts / workspaces** — a developer works across multiple accounts and runs `login` a second time; either support named profiles (e.g., `--profile staging`) or overwrite the stored token with a clear confirmation, never silently accumulate credentials.

## Common Mistakes

- Storing the auth token in a plaintext file instead of the OS keychain
- Writing any output to stdout in JSON mode that is not valid JSON (update notices, interactive prompts, progress indicators)
- Not exiting with a non-zero code on errors — scripts cannot detect failures
- Letting prompts block CI pipelines because `--yes` / `--force` flags were not implemented
- Giving the CLI token the same permissions as a full developer API key
- Not revoking the token server-side on `logout` — only removing it from the keychain
- Forgetting to handle `426 Upgrade Required` — produces a confusing crash instead of an actionable message
- Running the update check synchronously and adding latency to every command
- Not redacting `Authorization` headers in debug output
- Using hardcoded Unix-style config paths that break on Windows
- Publishing to npm with a long-lived classic npm token instead of trusted publishing via OIDC
- Not testing the `--json` output by piping it through a JSON parser in CI
