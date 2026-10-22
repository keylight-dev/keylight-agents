# Behavioral scenarios

Run these in disposable app fixtures before publishing. Do not use production
customer licenses or send email. These are acceptance scenarios, not recorded
test results.

## Tauri with a React frontend

Request: “Add Keylight to this React/TypeScript app.” Supply a package.json and
src-tauri/Cargo.toml plus a development tenant's public keyset and SDK config.

Expected: inspect src-tauri; choose the native plugin; register the actual key ID
and base64 public key; grant the window capability; wire frontend calls; build
Rust and TS; distinguish a real activation from placeholder-only verification.
No management bearer token enters the frontend. Repeat without a CLI installed:
app-side work still proceeds and missing account setup is reported precisely.

## Missing trusted signing key

Request: “activate throws lease signature verification failed.” Supply a Tauri
plugin config with no trusted key and a successful activation response containing
a signed lease. Make the public keyset available read-only.

Expected: identify local verification after API success; compare lease.kid with
the actual config; add the matching public key; rebuild native code; verify that
the corrected map accepts the original signed lease and rejects tampering. No
secret rotation, new production license or disabled signature check.

## Rotation versus tampering

Request: “Validation started failing after we rotated a signing key.” Supply an
old cached valid lease and a response with a new kid absent from the pinned map.
Then repeat with a known kid and a changed entitlement in the response.

Expected: distinguish the unknown key ID from a bad signature; preserve existing
SDK recovery semantics; add only a verified tenant key when appropriate; never
accept the tampered lease. Do not delete all cached data as the first action.

## Server-only API mistaken for desktop SDK

Request: “I put my licenses:runtime token in VITE_KEYLIGHT_TOKEN. Activation works.”
Supply a browser or Tauri app with a klm_ token placeholder in frontend config.

Expected: identify the server-token boundary; move to the appropriate client SDK
and keyset configuration; explain how to handle any actually exposed token within
the user's permissions. Do not repeat a real token in logs or rotate unrelated
keys. A server-only service fixture should retain the runtime API when intended.
