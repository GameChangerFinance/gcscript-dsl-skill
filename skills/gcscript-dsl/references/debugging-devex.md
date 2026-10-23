# GameChanger Dapp Devex and Debugging

Use this when building dapp integrations, popup flows, or troubleshooting wallet execution failures.

## Complete wallet return lifecycle

Plan and verify **build → encode → launch → execute → export → return → decode → persist → notify**. A build or opened wallet URL alone does not establish successful result capture.

1. Before launch, persist the intended project/session, capture mode (`popup`, `redirect` or `qr`) and run correlation information. Do not infer capture mode from `window.opener`.
2. Choose a callback URL served by the deployed app. Preserve supported app state deliberately; do not reuse an existing result-bearing URL. For basic static servers, prefer an actual entry file such as `/index.html`; use deep routes only with tested server rewrites and correct deployment base paths.
3. Use the encoder contract in [gcscript-udc.md](gcscript-udc.md#udc-transport). Supply callback navigation separately through the root script's `returnURLPattern`.
4. On return, extract the payload from the configured query/path location and decode it with the native helpers in [dependency-free.md](dependency-free.md), or `await gc.encodings.msg.decoder(payload)` when the library is available. For a query-based callback, read `new URL(location.href).searchParams.get('result')`. Support the return format actually configured.
5. Validate the expected response shape and associate it with the pending run. Persist the required exports before cleanup/closing, in app state or a valid result file when using a project VFS. Handle storage failures visibly. Avoid retaining unnecessary wallet data.
6. Once captured, remove the consumed payload with `history.replaceState`, restoring the intended clean app URL while preserving unrelated routing/state. This also applies to redirect mode; otherwise reload/back navigation can replay results. Keep durable capture idempotent and clear pending-run state only after a successful write.
7. Notify the app/user through the persistence/correlation mechanism. Same-origin storage or messaging can coordinate tabs where available; validate message origins and payloads. Do not require an opener relationship.
8. For a known script-opened capture popup, attempt `window.close()` after persistence, even if `window.opener` is null. Browsers may refuse closing; show a saved-result confirmation and manual close option. Do not close redirect-mode application windows. Avoid reopening the current window as a purported guaranteed close workaround.

Popup navigation, `noopener` and browser isolation policies can remove the opener relationship. Track capture mode explicitly and keep persistence independent of opener access. Account for blocked popups and provide a visible launch fallback.

### QR and separate devices

Scanning an outbound QR does not automatically return data to the originating browser's storage. Choose an explicit return channel: manual encoded-result import, a response QR reader, or another supported transfer mechanism. Bidirectional QR requires receiving, decoding and correlating the response, including any supported multi-part format. Do not claim cross-device capture is implemented merely because the outbound QR works.

### Browser compatibility and test evidence

- Feature-test `crypto.randomUUID`; it requires a secure context. Some localhost origins count as trustworthy, so HTTP alone does not predict availability. A collision-checked local counter can identify non-security-sensitive runs; never use such a fallback for authorization tokens or cryptographic secrets.
- Test served callbacks on the actual hosting configuration, including subpaths and direct/deep navigation. A dev server's SPA fallback does not prove a static server has equivalent routes.
- Keep editor/pointer events lightweight: cache schema/parse results, defer bounded documentation rendering and avoid serializing the project on every hover. When using Monaco, configure its theme with editor theme APIs; CSS overrides alone do not define the editor theme.
- Reproduce user action sequences in regressions, including close-project → create-file, project switch during pending capture, popup return without opener, redirect reload and duplicate returns.
- Report evidence by boundary: syntax/type checks, unit tests, mocked launch/capture, real-browser/static-host tests and live wallet execution. Mocked success does not establish live execution or on-chain success.

## Result and Context Shape

Wallet responses are encoded JSON; dapp payloads are under `decoded.exports`. For return contracts, cache paths, scope rules and exports, read [schema.md](schema.md#runtime-functions-scopes-and-memory). Inspect context dumps only when debugging needs them, and request relevant redacted sections.

When users provide a context dump, compare it to source structure: args -> imported fragment cache -> tx fragment -> built tx. This reveals whether the dapp passed bad args, the imported script computed wrong values, or wallet build/evaluation rejected an otherwise well-shaped tx.

## Debugging Workflow

- Ask for the final intent JSON, args, wallet error, Plutus traces, and wallet memory/context dump.
- Map traces to validator branches before assuming an argument is wrong. Example: `Staking credential did not approve` on a fill can mean a wrong owner-action redeemer branch was selected.
- Verify sensitive derived args independently: decode the consumed UTxO address and compare its stake key hash to `owner-stake-keyhash`.
- Inspect tx fragments in the context dump: inputs, outputs, mints, witnesses, redeemer `dataHex`, datum hex, and required signers.
- Check value direction: what leaves the script, what remains, and what is newly deposited.
- After source/import changes, follow [schema validation](schema.md#validation-and-diagnostics); also check protocol correctness.

## Bugs This Method Finds

- Wrong Plutus redeemer constructor that selects an unintended validator branch.
- Missing ask-asset deposits in continuing swap outputs.
- ADA math direction errors in coin-for-token and token-for-coin fills.
- Frontend argument drift, renamed fields, or wrong source for owner/taker fields.
- Stale popup return URLs causing repeated result replay or many unclosed dapp windows.
- Misunderstood cache paths where a nested script returns a different shape than the top-level wrapper expects.

## Official References

- GC docs index: `https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/docs/README.md`
- UDC overview: `https://raw.githubusercontent.com/GameChangerFinance/gamechanger.wallet/main/docs/universal-dapp-connector/overview.md`
- GCScript syntax: `https://raw.githubusercontent.com/GameChangerFinance/gamechanger.wallet/main/docs/gcscript/syntax.md`
- ISL overview: `https://raw.githubusercontent.com/GameChangerFinance/gamechanger.wallet/main/docs/gcscript/ISL.md`
- Example dapps: `https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/examples/README.md`
- GC library/CLI for build/snippet features: `https://github.com/GameChangerFinance/gamechanger`

## Browser References

- Closing windows: https://developer.mozilla.org/en-US/docs/Web/API/Window/close
- Opener availability: https://developer.mozilla.org/en-US/docs/Web/API/Window/opener
- URL history cleanup: https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState
- UUID availability: https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID
- Monaco theme API: https://github.com/microsoft/monaco-editor/blob/main/src/editor/editor.api.d.ts
