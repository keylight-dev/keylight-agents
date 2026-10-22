---
name: debugging-licensing
description: Diagnose an existing Keylight integration when activation, validation, entitlements, device limits, trials, or offline leases behave incorrectly. Use for errors such as lease signature verification failed; do not run a broad audit when a focused diagnosis is enough.
---

# Debug Keylight licensing

Locate the failing boundary, reproduce it with the smallest relevant evidence,
and fix that boundary. Do not assume an SDK signature error means the API
rejected the license or that the user needs to rotate a key.

## Establish what actually failed

Inspect repository instructions, dependency versions, initialization code, the
caller, and the exact error. Separate an HTTP failure, a success response with
`activated: false`, and a local verification error after `activated: true`.
For Tauri, determine whether the frontend calls the Rust plugin or the standalone
JS SDK. Capture the server status and response shape with credentials and license
keys redacted. Do not log raw tokens, SDK keys, private keys, or customer records.

Trace only as far as necessary along this chain:

payment → customer → license → entitlement → activation → device → lease → local verification

Use the installed SDK's implementation and matching official docs as the source
of truth. Start at https://docs.keylight.dev/ and the SDK's linked public repository.
Error wording can differ by release; an older SDK may combine several causes.

## Match the symptom to the boundary

| Symptom | Evidence to check first |
| --- | --- |
| Permission denied by Tauri before a request | Plugin registration and the intended window's `keylight:default` capability |
| API 401/403 | Base URL, tenant/product IDs, SDK-key header or server token scope; inspect the response before guessing |
| `activated: false` | Exact server error, key format/prefix, license status, product and device limit |
| Missing trusted keys | Actual map passed to the client; failed fetch silently replaced with an empty map |
| Unknown signing key ID | Returned `lease.kid` versus the configured map; tenant mismatch, omitted key or rotation |
| Known key, bad signature | Correct base64 public key and an unchanged returned lease; SDK/server format compatibility |
| Valid lease but feature locked | Lease entitlements, exact feature name, fallback/trial state, state hydration and refresh timing |
| Offline restart fails | Persistent store, embedded keys, device identity, expiry, offline window and device clock |
| Purchase/upgrade missing | Provider event, webhook delivery, license match and entitlement refresh; avoid another activation as the first fix |

## Diagnose signing-key failures precisely

Read the tenant's unauthenticated keyset at
`https://api.keylight.dev/{tenantId}/.well-known/keylight-keys`.
Find the entry whose `kid` matches the returned lease's `kid` and compare its
base64 `public_key` with the configured key. For a fresh integration, the primary
key is identified by `primary_kid`; existing leases may use another retained key.
Never substitute a key from another tenant, use the SDK key as the public key,
or assume that a key fetched successfully was passed into the client.

The Tauri config must carry `.trusted_key(kid, public_key_base64)` before plugin
initialization. The plugin does not fetch keys automatically. Check all config
construction paths, then rebuild and restart the Rust app after fixing the map.

If the key ID is known and verification still fails, inspect the original lease
before any app-side transformations. The SDK reconstructs the signed `v3` payload,
including sorted entitlements, using the contract at
https://docs.keylight.dev/security/ed25519-leases/.
Use the SDK verifier/conformance fixtures; do not introduce custom cryptography.
Expiry and clock checks are distinct from a cryptographic signature failure.

Never fix verification by disabling it, trusting `activated: true` alone, changing
the signed payload, or blindly fetching a new trust root at every launch. Product
settings verification (`requireSignedConfig`) is a separate check and cannot repair
a missing lease-verification key.

## Use live account tools when available

The CLI is optional. With an authenticated CLI, inspect the relevant command help
and read only the product, license, device or webhook records needed for this
failure using `--json`. Reuse existing auth; do not ask the user to paste a
management token into chat. Without account access, fix reproducible app-side
problems and identify the exact remaining record or setting to inspect.

Changing account configuration, deactivating devices, rotating secrets, issuing
licenses or replaying a payment webhook must fit the user's authorized task.
Do not use those actions as speculative diagnostics. Respect browser confirmation
gates and show their URLs to the human; do not automate approval. Avoid retry loops
that consume device seats or send customer messages.

## Verify the correction

Reproduce the failing case, apply the smallest correction, and rerun it. For key
handling changes, also prove that a valid signed lease is still accepted and an
unknown key or tampered lease is rejected without storing new license data.
Preserve the SDK's existing last-known-good/offline recovery behavior; an unknown
key ID during rotation is not the same as a known key with a forged signature.

Report the cause with file/response evidence, the correction, and checks actually
performed. If a live keyset or account could not be read, say which conclusion
remains unverified rather than presenting a likely cause as confirmed.
