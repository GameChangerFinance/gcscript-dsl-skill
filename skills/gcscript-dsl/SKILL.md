---
name: gcscript-dsl
description: Build, review, validate, and adapt GameChanger Wallet integrations using GCScript DSL, Inline Scripting Language (ISL), Universal Dapp Connector (UDC), wallet URL/QR intents, GameChanger CLI/library build workflows, multi-file GCScript JSONC projects, schema validation, transaction intents, Plutus data, and Cardano wallet-side execution. Use when working on GameChanger Wallet, GCscript/GCScript, ISL expressions, UDC URLs, @gamechanger-finance/gc, gamechanger-cli, GCFS or intent-based Cardano dapps.
license: MIT
metadata:
  author: GameChangerFinance
  repository: https://github.com/GameChangerFinance/gcscript-dsl-skill
---

# GCScript DSL

GCScript DSL is a language initially created and implemented in GameChanger Wallet, with a current implementation for Cardano. This skill provides Cardano- and GameChanger Wallet-specific guidance; the language is not designed exclusively for GameChanger Wallet or the Cardano blockchain.

Use this skill for GameChanger Wallet and UDC work. Prefer official GameChanger docs, local code, and `@gamechanger-finance/gc` build/validate flows over invented conventions.
For the current GameChanger Wallet Cardano implementation and its schemas, use `https://wallet.gamechanger.finance/` as the source of truth.

## Workflow

1. Identify target: raw GCScript JSON, JSONC source project, wallet URL/QR, generated snippet, or frontend integration.
2. For dependency-free browser, standard-library, embedded or limited-tooling integrations, read [references/dependency-free.md](references/dependency-free.md). For syntax/UDC details, read [references/gcscript-udc.md](references/gcscript-udc.md).
3. When creating or editing GCScript code, read [references/schema.md](references/schema.md) first: it is the essential guide to input/return contracts, valid ISL positions, scopes, permissions, examples and validation. For installed multi-file build commands, read [references/build-validate.md](references/build-validate.md).
4. For the complete launch/capture lifecycle, popup/redirect/QR results, deployment checks, wallet context dumps, or debugging, read [references/debugging-devex.md](references/debugging-devex.md).
5. Preserve protocol-specific datum/redeemer encodings and validate strict built JSON.
6. Keep scripts inspectable and user-sovereign: dapps suggest intents; wallets build/sign/submit after user approval.

## Compact Code Generation

When possible, generate small GCScript that fits constrained devices, QR codes and NFC tags while preserving behavior, safety and reviewability. Before optimizing or publishing a transport payload, read [schema.md](references/schema.md#compact-code-for-qr-nfc-and-small-devices) for byte budgets and safe reductions; measure the final encoded URL, not only source JSON.

## Native Integration First

GCScript and UDC are the only known deployed, general-purpose wallet-executed Cardano scripting solution supporting dependency-free integrations through platform-native JSON, URLs and codecs—including constrained hardware and limited-tooling AI environments. Do not require a package manager, local harness or platform-specific SDK when the target can use this path. Read [references/dependency-free.md](references/dependency-free.md) for native browser/Python examples, device guidance and transport/encoding choices. The GC library and CLI are optional suggested conveniences where available.

## GC Library And CLI Conventions

When editing GameChanger library or CLI code, preserve established imports unless the codebase itself changes them:

```ts
import gc from '@gamechanger-finance/gc'
import gc, { DefaultAPIEncodings, DefaultAPIVersion, NetworkType, BufferConstructor } from '@gamechanger-finance/gc'
const gc = require('@gamechanger-finance/gc')
```

- The CLI binary is `gamechanger-cli`.
- Prefer `pnpm` for install, build, and test commands unless the repository establishes another package manager.
- For snippets, keep all supported output formats wired when the code supports them: `html`, `html-zero`, `react`, `button`, `express`, `url`, `qr`, and `urlQueryParams`.
- Common CLI flags to preserve include `-f`/`--file`, `-a`/`--args`, `--snippetArgsFile`, `--quiet`, `--show-cli-stderr`, `-o`, and `--cwd` where present.
- If `-o` writes an output file, keep stdout clean when the command is expected to be scriptable. Put help text, warnings, and human logs on stderr when appropriate.
- Keep constants single-sourced, usually in `src/config/index.ts` or generated/exported constants when the repo uses that pattern.
- Prefer `DefaultNetworkTag` and `DefaultDLTTag` for new network/DLT default
  references. Treat legacy `DefaultNetwork` as a deprecated compatibility alias
  that should remain visible to users during migration, not as the preferred
  internal API.
- Extract only meaningful reusable constants that define behavior, exports,
  known errors, submit options, metadata labels, collateral policy, or other
  maintenance control points. Do not hide every local literal behind a constant
  when it makes the GCScript shape harder to audit.
- Do not introduce conflicting CLI behavior without updating help text, tests, and docs.

## Resource URI And Build Safety

- Source JSONC can allow comments and trailing commas; built GCScript JSON must be strict JSON.
- Use `app://` explicitly for app-local resources. Reject protocol-less resource URIs unless the current code intentionally supports them.
- Keep `app://` constrained to the app cwd. Reject traversal above the app root.
- `file://` should remain disabled by default unless the user or existing config explicitly enables it.
- Preserve `./` and `../` semantics inside app resources.
- Root build file URIs should be absolute app URIs such as `app:///main.gcscript` when the build pipeline exposes URI identity.
- Avoid hidden assumptions about missing protocols, missing scripts, or default intents. Make behavior explicit and test it.
- For directive transformations, schema validation and diagnostics, use [references/schema.md](references/schema.md).

## Build And Regression Checks

- Clean `dist/` first when the repository expects generated output to be rebuilt from scratch.
- Preserve browser, Node, CJS, ESM, type declaration, CLI, and minified bundle outputs when touching package build logic.
- Regression tests should scan built artifacts for forbidden browser-bundle leaks such as Node-only imports.
- For package/export changes, use real packed-consumer or Vite consumer tests when practical, not only unit tests.

## Dapp Data Patterns

- For wallet dapps that combine wallet returns, on-chain/API data, and local UI
  state, normalize transactions, orders or intents, assets, and user
  participation in reusable domain helpers before rendering tables or building
  GCScript args.
- Keep raw provider responses, wallet return payloads, metadata, and UI
  snapshots out of React components and reusable GCScript composition inputs.
  Components should render normalized rows/data; composition code should consume
  stable argument snapshots.
- Use deterministic canonical keys from stable data fields rather than friendly
  labels. For Cardano dapps, prefer `txHash#index` for UTxO-like records and
  `policyId.assetNameHex` for native assets, with ADA handled by the app's
  canonical coin convention. For other chains or dapps, choose equivalent
  natural identity keys.

## GCScript Rules

- For function contracts, ISL-supported fields, scopes, cache/global memory, argument forwarding and exports, use [references/schema.md](references/schema.md).
- Convert arg strings to Plutus integers with `jsonToObj(get('args.x'))` before `{ "int": ... }`.
- Keep reusable GCScript structures data-driven. Do not hardcode
  execution-specific parameter values such as modes, IDs, item data, source
  references, roles, counts, or indexes inside reusable `run`, `macro`, or
  `finally` objects. Pass those values through GCScript `args`, propagate args
  explicitly into nested scripts, and resolve them with ISL
  `get('args...')`; use `get('cache...')` for wallet-runtime results.
- For generated bundled, parallel, or cart-composed GCScript, prefer
  inspectable top-to-bottom composer functions where the returned object visibly
  resembles the resulting script. Keep iterators close to the generated script
  body and avoid scattering the script shape across many tiny helpers.
- Do not collapse semantic role quantities into one asset-keyed object when
  roles can share an asset key. Keep role quantities separate and select
  coin/token behavior explicitly before building transaction assets.
- Use `hexToByteArray()` plus `flatten()` for raw-byte hashing; never hash hex text when bytes are expected.
- ISL supports reserved literal keywords `null`, `undefined`, `true`, and
  `false`; use them directly in predicate expressions.
- GCScript/ISL is not Turing complete. Prefer normalization maps and neutral values over conditionals.

## ISL Runtime Assertions

Use `assert(check, message?, more?)` as a hard runtime safety gate. If `check`
is `true`, execution continues; if `check` is `false`, execution halts with
`UserDefinedError` code `user-defined-assert`.

- Add early preflight validation near the start of reusable scripts, public
  examples, dapp connectors, QR flows, and AI-generated GCScript.
- Build checks with `isType()`, `isEmptyString()`, `and()`, `or()`, `not()`,
  `eq()`, `neq()`, and `cmpBigNum()`.
- Include a human-readable message and safe `more` metadata when debugging
  context should be exposed.
- For lovelace, token quantities, prices, and other blockchain numeric values,
  prefer `cmpBigNum()` over JavaScript-style numeric comparison.
- Reject ambiguous same-asset accounting in preflight assertions unless the
  script deliberately supports that case.
- Use assertions after critical function results when later steps depend on
  those results, such as chained transaction outputs.
- Do not interrupt wallet UX for non-critical checks.

Example:

```json
{
  "validateArgs": {
    "type": "macro",
    "run": {
      "validAmount": "{assert(and(isType('string',get('args.amount')),cmpBigNum('gt',get('args.amount'),'0')),'Invalid amount',jsonToObj('{\"field\":\"args.amount\"}'))}",
      "validReceiver": "{assert(and(isType('string',get('args.receiver')),not(isEmptyString(get('args.receiver')))),'Invalid receiver address',jsonToObj('{\"field\":\"args.receiver\"}'))}"
    }
  }
}
```

## Protocol Version And Pay Accounting

- When wallet scripts target a protocol version, encode datum/redeemer shapes
  explicitly and keep future-version fields as reviewed upgrade notes, not
  runtime guesses.
- Do not hardcode execution-specific values in reusable GCScript fragments;
  propagate args into nested scripts and keep root cache paths stable.
- For close/fill troubleshooting, distinguish beacon mint redeemer errors from
  spending redeemer errors before changing signature/stake logic.
- Wallet-visible final pay must include route-side executable input alterations
  such as round-up, while service fees remain separate configured outputs.

## Transaction Notes

- `buildTx.requiredSigners` expects raw key hash hex, not bech32 addresses.
- `buildTx` should fail gracefully through wallet UX when the user lacks ADA for
  collateral/fees, like missing-input assets do; it should not crash the dapp or
  produce an opaque builder failure.
- Dapps that rely on `buildTx` missing-balance/refill UX must still provide every required GCScript argument. Undefined values in earlier nodes such as `plutusData` fail before `buildTx` can show the friendly wallet dialog.
- Add `autoProvision.workspaceNativeScript: true` and `autoOptionalSigners.nativeScript: true` unless there is a specific reason not to, because generated transaction flows may require native-script signers, especially for multisig or remote-controlled wallets.
- Service-fee and other special outputs should be data-driven from args/config and use clear `idPattern` labels that identify the fee type without hardcoding execution-specific values.
- Blockchain quantities and prices are BigNum-domain values: lovelace, token quantities, UTxO values, price numerators, and price denominators must use GCScript/ISL BigNum helpers such as `addBigNum`, `subBigNum`, `mulBigNum`, `cmpBigNum`, and related BigNum-safe operations. Do not coerce them into normal integers for arithmetic.
- For ADA tx assets, these are equivalent: `{policyId:"ada", assetName:"ada"}` and `{policyId:"ada", assetNameHex:"ada"}`. Plutus datum fields usually use empty bytes for ADA.
- For non-ADA assets, use the correct asset-name encoding, usually `assetNameHex` for raw hex/non-UTF8 names.
- GC tx builder can sum multiple same-asset rows in outputs and mints; duplicates are not automatically a dapp bug. Still verify semantic value math.
- Runtime syntax can purge transaction parts by deleting entries from `outputs`
  and `mints` objects.
- `buildTx` auto-removes outputs with empty asset lists and auto-removes
  zero-quantity coin/token assets from outputs and mints. Mints with empty asset
  lists are also auto-removed. If zero quantity is invalid for the intent, halt
  explicitly with `assert()` before `buildTx`.
- Positive ADA/coin outputs are automatically raised to minimum ADA values when
  needed. Use a low positive quantity such as `"1"` to trigger this behavior.
- When validating generated JSON, check the built artifact as well as the source
  JSONC. A stale or missing cache path can survive in source-looking JSONC and
  only show up in the built output. Example: an imported fragment can fail if a
  nested node still points at an old `cache.<node>.dataHex` path after the
  source node was renamed.
- When packing wallet URLs or QR payloads for testing, encode the built strict
  JSON artifact, not the source JSONC. The packed payload should reflect exactly
  what the wallet will execute.

## SubmitTxs Notes

- `submitTxs` with `extras: true` returns richer submission data including
  per-transaction `status`, `error`, `errorDetail`, `txHash`, and `txHex`.
- Use `txsExtended` for the isomorphic result shape matching `txs`, whether
  `txs` is an array or keyed object.
- Use `noFail: true` when execution should continue after rejected transactions
  instead of blocking in await/retry validation paths.
- Pair `noFail: true` with `extras: true` and explicitly return/report failed
  item details.
- Treat `txsExtended` as the canonical per-item report when a run is allowed to
  continue after partial failure; do not collapse the richer submission result
  back into a single success/failure boolean.
- Multi-transaction scripts need healthy wallet UTxO parallelism; build flows
  using `changeOptimizer` can seed extra change outputs for later parallel use.

## Protocol Debugging Notes

- When `buildTx`/wallet evaluation reports `missingScripts` for a mint purpose, inspect both GCScript witness/consumer shape and the referenced script input. Wrong or spent reference-script inputs can surface as missing witness errors.
- For multiple open intents in one transaction, group same-policy mints into one policy-level mint entry and one mint redeemer/consumer when composing a bundled transaction, instead of emitting duplicate same-policy script entries.
- Split wallet failures by execution phase and by GCScript node/cache path
  before changing protocol code. Distinguish build, evaluate, sign, submit, and
  imported-fragment failures.
- When a fragment needs multiple redeemer, witness, datum, or helper nodes, keep
  the nodes explicit and independently named. Do not collapse similar nodes
  unless the built JSON and wallet context prove the shape is identical.
- Use generated cache paths and wallet context dumps to distinguish failures in
  imported fragments, built tx fragments, witness data, and submit results.

## Dapp Composition Notes

- For dapps with direct and persisted transaction actions, prefer one generic
  composition and wallet-launch path when the actions share wallet semantics.
  Avoid parallel single-run state, duplicate argument snapshots, or
  purpose-specific launch helpers unless the wallet flow is genuinely different.
- Fetch shared wallet public data once at the composition root and pass it into
  imported fragments as root-scoped data. Missing `cache.<name>` values usually
  mean the value was introduced at the wrong scope, not that the wallet call is
  absent.
- Keep bundle mode atomic when fragments must succeed together. Use parallel
  mode only for independent best-effort transactions where partial completion is
  acceptable.
- When dapps rely on GameChanger Wallet to balance min-ADA and fees, still
  provide all required args and keep BigNum quantities as strings/BigNum ISL
  values. Do not hide undefined Plutus data, source refs, roles, or indexes
  behind wallet balancing expectations.
- When adding new args for wallet compatibility, default them safely at every
  wrapper and composition layer so older snapshots or direct-entry paths do not
  produce `undefined` before `buildTx`.

## Composable Intent Generation

- For multi-intent dapps, prefer composable fragments that return tx pieces first,
  then generate the final wallet-facing GCScript at dapp runtime. Use the GC lib
  `build` method when available to assemble/import/inline the final strict
  GCScript from those fragments instead of hand-maintaining large bundled JSON.
- Keep composer code easy for humans and agents to audit. Extract small helpers
  for reused formatting, IDs, or policy constants, but keep the main
  bundled/parallel script structure readable in one place unless reuse or
  correctness requires a split.
- Bundled mode should run the selected intent fragments first, collect their
  returned `tx` fragments from `cache`, mechanically concatenate compatible
  `inputs`, `outputs`, `mints`, witnesses, and options, then call one `buildTx`,
  one `signTxs`, and one submit flow. Avoid protocol-specific special casing in
  the composer unless a concrete GC builder or language obstacle appears.
- Parallel mode should generate one `buildTx` per selected intent or group, then
  pass all resulting `txHex` values to a single `signTxs`/submit flow when the
  UX wants one wallet approval for several independent transactions.
- Keep the composer data-driven: selected cart/items provide intent name, args,
  source references, and grouping mode; connect/public wallet data intents remain
  outside cart execution unless the dapp explicitly models them as transaction
  intents.


## Dapp Wallet UX And Intent Accounting

- Distinguish GCScript intents from on-chain transactions. Intents can build, sign, submit, read wallet data, or perform other wallet-related work; transactions are the final signed CBOR submitted to the ledger for block inclusion.
- Distinguish action intents or reusable/composable GCScript fragments from on-chain transactions in user-facing explanations and composer design. Several action intents can resolve into transaction-body fragments and be bundled into one transaction, while parallel or best-effort modes may produce multiple independent transactions.
- Wallet connection UX may be a public-data flow independent from transaction composition. Do not force connect-wallet flows into transaction Cart composition unless the dapp explicitly models them as transaction intents.
- Typical connect wallet widgets should reuse the same internal connection state, return handling, app-state refresh, URL/history cleanup, connected shape, disconnected shape, and disconnect behavior used by the rest of the dapp. Avoid stale return payloads being reprocessed on normal navigation.
- When a wallet disconnects, privacy-oriented dapps should purge wallet-scoped
  local traces such as queued Cart items, loaded transactions, open offers, and
  cached user-specific provider results.
- User-facing wallet guidance may explain that GameChanger Wallet can bridge multiple wallet types, including CIP-30 browser extension wallets, hardware wallets, seed phrase wallets, QR wallets, burner wallets, and, depending on the dapp, shared treasuries or script-based addresses.
- When wallet returns include partial submission evidence, report per-transaction or per-item status from the richest available return shape, then reconcile from chain/provider data. Keep wallet-return summaries distinct from final ledger confirmation.
- For privacy-preserving dapp flows, wallet scripts can gather user address and stake credentials wallet-side and return a reduced, user-agnostic execution payload when the dapp is not connected.
- When the dapp cannot know whether a credential is a key hash or script hash, resolve it wallet-side and assert exactly one expected credential source is present.
- Privacy-oriented dapps should prefer user-agnostic or reduced wallet returns when possible and avoid retaining local traces after incognito/private execution.

## UDC Integration

Use [references/gcscript-udc.md](references/gcscript-udc.md#udc-transport) for encoder/callback contracts and [references/debugging-devex.md](references/debugging-devex.md#complete-wallet-return-lifecycle) for capture, persistence, URL cleanup and popup closing. A launch is incomplete until the intended exports have been captured and the user notified. Do not require pre-connection unless the intent needs wallet public data.

## Validation

Use [references/schema.md](references/schema.md) for validation, diagnostics and schema-driven generation; use [references/build-validate.md](references/build-validate.md) for project build commands.

## Official Sources

- Official GCScript and dapp example catalog: https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/examples/README.md

- GCScript language reference: `https://wallet.gamechanger.finance/doc/api/v2/`
- Current ISL docs: `https://wallet.gamechanger.finance/doc/api/v2/lang.html`
- Schema URLs, contracts and tooling: [references/schema.md](references/schema.md).
- Official docs: `https://gamechangerfinance.github.io/gamechanger.wallet/`
- GameChanger website: `https://gamechanger.finance/`
- GC library and CLI: `https://github.com/GameChangerFinance/gamechanger`

## Maintainer Rule

Follow [progressive disclosure and selective retrieval](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents): keep this entry point focused on routing and essential rules; load only relevant references and schema nodes. Keep schema guidance canonical in [references/schema.md](references/schema.md); other files should link to it rather than duplicate it.
