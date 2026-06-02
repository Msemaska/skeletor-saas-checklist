# IDE Extensions

---

## What It Is

An IDE extension is a thin client that surfaces your product's functionality directly inside a developer's editor. It is a distribution channel and a UI layer — not a second product. The extension calls your existing public API (phase-4-public-api/01-api-foundation.md) for all data and business logic. The editor is just the surface. Done correctly, the extension dramatically reduces context-switching for developer-focused users and becomes a high-retention touch point.

## Scope and Sequencing

Ship VS Code first. VS Code has the largest installed base among developer tools by a wide margin and the lowest barrier to publish. JetBrains (IntelliJ, WebStorm, PyCharm, GoLand, and the rest of the IDE family) is the second target — a single JetBrains plugin covers all JetBrains IDEs simultaneously. Do not build for both in parallel on the first release; finish and publish the VS Code extension, then port to JetBrains.

The extension does not contain business logic. It renders UI, calls the public API, and stores a token. Any feature that requires logic belongs in the API, not the extension. This means feature changes deploy server-side without requiring an extension update or marketplace re-review.

## Architecture

The extension is a thin client over the public API. It has three concerns:

1. **Authentication** — sign the user in once, store the token, refresh it transparently.
2. **UI** — sidebar panels, status bar items, tree views, webviews, and command palette commands that mirror the features exposed by the public API.
3. **API calls** — every action calls the public API using the stored token. The extension never touches the database directly.

The extension communicates with one endpoint: your public API base URL. It does not embed any secrets. The only credential it holds is the user's access token, which the user consented to by signing in.

## Authentication

Use the OAuth browser/device flow. When the user runs a "Sign In" command, open a browser to your product's standard OAuth authorize URL (the same one used by the public API's OAuth flow — phase-4-public-api/01-api-foundation.md). After the user authenticates in the browser, the auth server redirects with a code that the extension exchanges for an access token and refresh token.

Store the access token and refresh token in VS Code's built-in `SecretStorage` API (not in extension settings, not in a local file, not in workspace state). `SecretStorage` delegates to the OS credential store — Windows Credential Manager on Windows, macOS Keychain on macOS, the Secret Service on Linux. On JetBrains, use the `PasswordSafe` credential store from the IDE's credential management API, which provides the same OS-level isolation.

Before every API call, check whether the access token has expired using the stored expiry timestamp. If it has, use the refresh token to obtain a new access token silently. The user should never be prompted to re-authenticate unless the refresh token itself has expired or been revoked.

Never request more OAuth scopes than the extension actively uses. Scope down to exactly what the extension needs.

## Telemetry

Collect telemetry only with the user's explicit consent. On VS Code, gate all telemetry on the `isTelemetryEnabled` API, which reflects the user's global telemetry preference. Do not implement a separate in-extension opt-in prompt — the platform-level preference is the consent signal. Emit no telemetry when `isTelemetryEnabled` is false.

Collect only what informs product decisions: which commands are run, which panels are opened, and error events. Do not log API request payloads, file contents, or anything that could contain user data.

On JetBrains, use the IDE's built-in `UsageTrigger` or `FUSEventLogger` for feature usage statistics. These respect the user's JetBrains data-sharing preference automatically.

## Versioning and API Compatibility

The extension declares a minimum API version it supports. Every extension release notes the minimum backend API version required. When the public API introduces a breaking change, the API must version the endpoint (phase-4-public-api/01-api-foundation.md handles this). The extension checks the API version on startup and shows a clear upgrade prompt if the running extension is too old to work with the current API.

Keep extension releases small and frequent. Smaller updates pass marketplace review faster and reduce the blast radius of any regression. Ship features server-side where possible to avoid requiring an extension update at all.

## Marketplace Publishing

**VS Code Marketplace:** Requires a free Microsoft account and a publisher identity created at marketplace.visualstudio.com. The review for new extensions takes 2–5 business days. Updates are reviewed in under 24 hours in most cases. The listing requires: extension name, short description (under 200 characters), detailed description (Markdown), icon (128x128 PNG), screenshots, a repository link, and a privacy policy URL. Submit at least one week before the planned launch.

**JetBrains Marketplace:** Requires a JetBrains account and registration as a vendor. Every new plugin and every update goes through manual review. The team targets 3–4 business days but does not guarantee a timeline. The listing requires: plugin name, description, screenshots, and a privacy policy. Any plugin that interferes with JetBrains licensing, subscription validation, or trial flows is rejected — do not touch those internal APIs.

Both stores require a privacy policy URL. Use the same `/privacy` page from the main site. The policy must disclose that the extension collects an access token, what it is used for, and how the user can revoke it.

## Providers

| | VS Code Marketplace | JetBrains Marketplace |
|---|---|---|
| **Audience** | Largest developer IDE installed base | IntelliJ, WebStorm, PyCharm, GoLand, and all JetBrains IDEs via one plugin |
| **SDK** | VS Code Extension API (TypeScript-native) | IntelliJ Platform Plugin SDK (Kotlin-native) |
| **Review time** | 2–5 days (new), under 24 hours (updates) | 3–4 business days, no guarantee |
| **Auth storage** | `SecretStorage` API → OS credential store | `PasswordSafe` API → OS credential store |
| **Telemetry control** | `isTelemetryEnabled` API | IDE data-sharing preference |
| **Publishing fee** | Free | Free |
| **Ship order** | First | Second |

## Security Considerations

**Secret storage:** Tokens must live in `SecretStorage` (VS Code) or `PasswordSafe` (JetBrains). Never store tokens in `settings.json`, workspace settings, or any file the extension writes to disk — these are plaintext and readable by any process on the machine.

**No secrets in the extension bundle:** The extension contains no API keys, no client secrets, and no service credentials. It holds only the user's access token, which they explicitly granted. The OAuth client ID (used to initiate the flow) is not a secret and may appear in the extension bundle, but the client secret never does — it lives server-side.

**Scope minimisation:** Request only the OAuth scopes the extension actively uses. Overly broad scopes increase the impact of a stolen token and trigger rejection from the marketplace review teams.

**Token revocation surface:** The product's API must support token revocation so that when a user signs out of the extension, the token is invalidated server-side. A user removing the extension should not leave a live token with no revocation path. The "Sign Out" command must call the revoke endpoint before clearing local storage.

**Webview content security:** If the extension uses a webview panel, set a strict Content Security Policy that disallows inline scripts and restricts resource loading to the extension's own assets and the product's API origin. Do not load arbitrary external content in webviews.

## Deliverables / Implementation Checklist

**VS Code Extension**
- [ ] VS Code extension scaffolded with TypeScript
- [ ] Publisher identity created on marketplace.visualstudio.com
- [ ] OAuth browser flow implemented: opens browser, receives callback, exchanges code for tokens
- [ ] Tokens (access + refresh + expiry timestamp) stored in `SecretStorage`
- [ ] Token refresh logic implemented: checks expiry before every API call, refreshes silently
- [ ] "Sign In" and "Sign Out" commands registered in the command palette
- [ ] Sign Out calls the API revoke endpoint before clearing `SecretStorage`
- [ ] API version compatibility check on extension activation
- [ ] Telemetry gated on `isTelemetryEnabled` — no telemetry emitted when disabled
- [ ] No tokens, secrets, or credentials in `settings.json` or any writable file
- [ ] Webview Content Security Policy configured (if webviews are used)
- [ ] Extension icon (128x128 PNG), short description, Markdown README, and screenshots prepared
- [ ] Privacy policy URL (`/privacy`) included in `package.json` metadata
- [ ] Extension submitted to VS Code Marketplace at least one week before launch

**JetBrains Plugin**
- [ ] JetBrains plugin scaffolded with Kotlin and IntelliJ Platform Plugin SDK
- [ ] JetBrains Marketplace vendor account created
- [ ] OAuth browser flow implemented using IDE browser-launch API
- [ ] Tokens stored in `PasswordSafe` credential store
- [ ] Token refresh logic mirrors VS Code implementation
- [ ] Sign In / Sign Out actions registered as IDE actions
- [ ] Telemetry respects IDE data-sharing preference
- [ ] Plugin listing assets prepared: description, screenshots, icon
- [ ] Privacy policy URL included in plugin.xml metadata
- [ ] Plugin submitted to JetBrains Marketplace

## What to Test

- Sign in flow completes and token is present in `SecretStorage` / `PasswordSafe` after successful auth
- Token refresh fires silently when the access token is expired before an API call
- Sign out clears `SecretStorage` / `PasswordSafe` and calls the server-side revoke endpoint
- No token or credential appears in `settings.json`, workspace state, or any file on disk after sign-in
- Extension activates cleanly when offline (no crash, graceful degraded state shown)
- Extension shows a clear message and disables API-dependent commands when not signed in
- API version mismatch is detected and an upgrade prompt is shown
- Telemetry events are not emitted when the user has disabled telemetry at the platform level
- Webview (if present) loads only from approved origins and blocks inline scripts
- Revoke endpoint is called when the user signs out from the extension
- Extension installs and activates correctly from the packaged `.vsix` (not just in extension development host)

## Edge Cases

- **Token expires while the editor has been open for days** — the refresh-before-call logic handles this: check expiry on every API call and refresh silently; never assume a token stored hours ago is still valid.
- **Refresh token itself expires** — detect the 401 from the refresh endpoint and prompt the user to sign in again; do not silently fail or loop.
- **Extension version too old for the current API** — on activation, fetch the API's minimum supported extension version and show a non-dismissible prompt to update from the marketplace if the installed version is below that threshold.
- **Marketplace review delays the launch** — VS Code and JetBrains reviews can take longer than expected; submit both extensions at least one week before the planned launch date and never announce a launch date before both are approved.
- **User has multiple editor windows open** — `SecretStorage` is scoped to the extension and shared across all windows; a sign-in in one window is reflected in others, and a sign-out must clear the shared storage, not just the local window state.
- **User is signed into multiple accounts** — the extension supports one active session at a time; switching accounts requires signing out and signing back in; do not attempt silent multi-account management in v1.
- **Working offline** — the extension must activate and render without making API calls on startup; defer API calls to user-initiated actions and show an offline indicator rather than crashing or showing unhandled error states.
- **Extension auto-updates while the user is mid-session** — VS Code applies extension updates on window reload; the token persists in `SecretStorage` across updates, so the user does not need to re-authenticate after an update.

## Common Mistakes

- Storing the access token in `settings.json` or workspace state — plaintext, readable by any process, instantly disqualifying from a security audit
- Building the JetBrains plugin before the VS Code extension is stable — splits focus, produces two half-finished surfaces instead of one polished one
- Embedding business logic in the extension instead of the API — every logic change then requires a marketplace update and review cycle
- Not implementing token refresh — tokens expire; users get silent failures hours into their session
- Not calling the revoke endpoint on sign out — leaves a live token with no invalidation path
- Requesting broad OAuth scopes "for future features" — triggers marketplace rejection and violates the principle of least privilege
- Shipping without an API version check — the extension silently breaks when the API introduces a change the old extension cannot handle
- Using webviews for everything — prefer native VS Code tree views, status bar items, and panels where possible; webviews carry more complexity, a stricter CSP requirement, and slower rendering
- Not testing the packaged `.vsix` before submission — extensions that work in development often fail as packaged builds due to bundling issues
- Announcing a launch date before marketplace approval is confirmed — both marketplaces can delay; never commit to a public date until the extension is approved
