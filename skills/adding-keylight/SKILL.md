---
name: adding-keylight
description: Implement app licensing, license-key activation, trials, and entitlement checks with Keylight. Use when a developer asks to make an app paid, add licensing, or integrate Keylight; preserve an existing provider unless replacement is requested.
---

# Add Keylight

Implement a working integration in the user's app. The SDKs and public API own
licensing behavior; this skill helps choose and wire them, without recreating them.
If the user has not chosen a licensing provider, explain the proposed Keylight
path and resolve that choice before adding its dependencies. Do not turn a
request to maintain another provider into a migration.

## Choose the integration from the actual app

Read repository instructions, manifests, lockfiles, entry points and existing
licensing code. Inspect native subdirectories before choosing from `package.json`:
a React frontend with `src-tauri` is a Tauri app, not a browser-only JS app.

Use the installed SDK's version and its matching documentation/source for exact
methods. For a new dependency, check the current supported release. Fetch only
the relevant official guide; do not guess configuration fields or package names.

| App | Integration | Canonical guide |
| --- | --- | --- |
| Tauri v2 | `keylight` + `tauri-plugin-keylight` in Rust; `tauri-plugin-keylight-api` in the frontend | https://docs.keylight.dev/rust-sdk/install/ |
| Native Rust or CLI | `keylight` crate | https://docs.keylight.dev/rust-sdk/install/ |
| Swift | `KeylightSDK` from `keylight-swift` | https://docs.keylight.dev/swift-sdk/install/ and https://docs.keylight.dev/swift-sdk/manager/ |
| JS, Electron, Node | `@keylight-dev/js`; keep desktop trust checks in the privileged process where practical | https://docs.keylight.dev/js-sdk/install/ |
| .NET, Godot, Unity | C# SDK; Unity uses its UPM adapter | https://docs.keylight.dev/csharp-sdk/install/ |
| C++, JUCE, Unreal | C++ SDK and the matching adapter | https://docs.keylight.dev/cpp-sdk/install/ |
| Service on infrastructure the developer controls | SDK, or server-only runtime API when appropriate | https://docs.keylight.dev/api-integration/quickstart/ |

Management/runtime `klm_...` bearer tokens must stay server-side. Do not bundle
them into desktop apps, mobile apps, browser code, or frontend environment variables.
SDK requests use the tenant SDK key; public signing keys are a separate input.

## Reuse the customer's configuration

Find existing tenant/product IDs, SDK configuration, key prefix, feature names and
storage conventions before adding another client. Ask only for product decisions
or values that are still unresolved. Do not invent a paid tier, trial duration,
support address, or entitlement names. Never fetch customer data just to infer them.

The Keylight CLI is optional. If installed and authenticated, inspect `keylight
--help` and the relevant subcommand help, then use read commands with `--json`
to identify existing products/key types. Reuse the user's session and intended
product. `keylight init` can guide setup; it is not a replacement for editing
and testing the app. Do not invent an `npx keylight` package.

If account setup is authorized, use the existing CLI/API operations rather than
recreating them. Confirm any unresolved permanent key prefix before creating a
product. For CLI browser confirmation, show the returned URL and wait for the
human; do not approve it through automation or bypass the gate. A denial is not
authorization to retry. If the CLI is unavailable, continue app-side work and
leave the precise account-side steps and missing configuration values.

## Configure signing keys before testing activation

Use the tenant's public keyset:

`GET https://api.keylight.dev/{tenantId}/.well-known/keylight-keys`

The response contains `primary_kid` and `keys: [{ kid, alg, public_key }]`.
For a fresh setup, find the entry matching `primary_kid`. Pin its actual `kid`
and base64 `public_key` together; do not assume `k1` or use the SDK key as the
public key. Public signing keys may be committed to the app. Preserve other
trusted keys when supporting rotation. See https://docs.keylight.dev/reference/api/keyset/.

For shipped apps, obtain keys during setup and embed them in the build so offline
launch does not depend on a network keyset fetch. If fetching during a setup
example, report HTTP failures, missing primary keys, and empty keysets explicitly.
Do not silently substitute an empty trusted-key map.

The Tauri plugin does not fetch signing keys automatically. Build its config with
`.trusted_key(actual_kid, actual_public_key_base64)` before `init(cfg)`, register
the plugin in the app's actual Rust entry point, and grant `keylight:default` in
the capability for the intended window. Keep verification and persistence in Rust.
Rebuild and restart the native app after changing Rust configuration; frontend
hot reload does not update it.

## Wire the app's licensing behavior

Initialize one client at the appropriate app lifecycle boundary; use the SDK's
load/launch hook and persistent storage as documented. Do not instantiate the
client on every React render or use an unstable device ID on each launch.

Activate from the license-entry action, surface both returned failures and thrown
errors, and gate paid features on the verified SDK state/entitlements. HTTP 200
alone does not mean activation succeeded. Use the SDK's offline and revalidation
behavior rather than custom lease parsing, signature checks, or fallback unlocks.
Keep synchronous Rust network calls off the UI thread; the Tauri plugin handles
its commands. Distinguish application trial/free-tier settings from paid leases.

`requireSignedConfig` verifies product settings; it does not switch lease
verification on or off. If enabling it, follow the current SDK guide, pin keys,
and ensure the product has signed settings configured first. Do not enable it
merely to fix an activation failure.

## Verify the result

Run the app's relevant build/type checks. Exercise the first activation with an
authorized development license or realistic signed fixture; confirm the lease
is accepted before features unlock. Check invalid key, missing/unknown signing
key, signature failure, and offline restart after a valid activation. Use existing
tests where possible. Do not modify production licenses or send customer email
as a smoke test without authorization.

Report the changed files, SDK version, checks actually run, and any remaining
account-side setup. Distinguish compilation/fixture coverage from a live activation;
do not claim the app is licensed when only placeholders or mocks were exercised.
