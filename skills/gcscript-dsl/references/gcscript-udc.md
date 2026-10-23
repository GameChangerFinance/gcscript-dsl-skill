# GCScript, ISL, UDC, and GameChanger Wallet

In its current Cardano implementation, GCScript is a JSON intent DSL interpreted by GameChanger Wallet. It is scoped, inspectable, permissioned, non-Turing-complete, and suited to URLs, QR codes, redirects, and local wallet execution.

## Schema-driven Coding and Scopes

Read [schema.md](schema.md) for function/ISL contracts, accepted ISL fields, scopes, argument forwarding, memory, exports and validation.

## ISL Notes

ISL field eligibility is defined in [schema.md](schema.md#isl-support-is-explicit). Useful functions include `get`, `join`, `addBigNum`, `subBigNum`, `mulBigNum`, `jsonToObj`, `strToHex`, `hexToStr`, `hexToByteArray`, `flatten`, `sha256`, and `getAddressInfo`.

Use `hexToByteArray` before `sha256` when hashing protocol byte preimages. Use `jsonToObj(get('args.x'))` for Plutus integer fields.

GCScript is not Turing complete. For dynamic asset cases, use object lookup maps and neutral sentinel values (for example `ada-ada: "0"`) instead of conditionals.

## Cardano Transaction Intents

`buildTx` builds transactions with outputs, inputs, mints, witnesses, withdrawals, certificates, metadata, validity fields, and options. Coin selection/change/collateral are wallet responsibilities unless overridden.

For Plutus:

- Use `plutusData` to serialize redeemers/datums from JSON.
- Inline datum output uses `datum: { datumHex: "..." }`.
- Do not add witness datum for Plutus V2 inline datum spends unless required.
- `requiredSigners` expects raw key hash hex values, not bech32 addresses.
- For ADA tx assets, `{ "policyId": "ada", "assetName": "ada" }` and `{ "policyId": "ada", "assetNameHex": "ada" }` are equivalent.
- For native assets, use the field matching asset-name encoding, commonly `assetNameHex`.
- The transaction builder may sum duplicate same-asset rows; duplicates are acceptable when intentional. Validate zero-quantity rows before relying on them.

## UDC Transport

UDC wraps GCScript into wallet-facing URLs or QR payloads. Dapp-to-wallet carries the intent; wallet-to-dapp return URLs carry exported execution results.

For native integration with no installed library, use [dependency-free.md](dependency-free.md). When the GC library is already available, use `await gc.encode.url({input: JSON.stringify(built), apiVersion: '2', network: selectedNetwork, encoding: 'gzip', ...options})` with supported values from the exact library version. `network` is an encoder option; the library normally produces the `networkTag` wallet URL parameter. Pass referral attribution as `refAddress`; it is not a callback URL. Configure callbacks with the root script's `returnURLPattern`. Inspect the documented bundle exports before adding separate encoding dependencies.

`returnURLPattern` is an absolute URL template. Preserve literal `{result}` when using explicit substitution; without it, the documented fallback adds a `result` query parameter. Supply raw URL strings in GCScript, not Markdown links or HTML-escaped `&amp;`. Ordinary query separators are not intrinsically disallowed by the schema. Validate the program, parse/test the resulting URL and verify callback payload extraction; schema validation alone does not prove valid routing or placeholder substitution. Do not accidentally percent-encode the placeholder while constructing the template.

For browser persistence, popup closing, redirect history cleanup and QR returns, load [debugging-devex.md](debugging-devex.md#complete-wallet-return-lifecycle). Keep Project terminology distinct from the wallet's Workspace concept.

## Sources to Prefer

- Official docs: `https://gamechangerfinance.github.io/gamechanger.wallet/docs/`
- UDC overview: `https://gamechangerfinance.github.io/gamechanger.wallet/docs/universal-dapp-connector/overview.html`
- Syntax: `https://gamechangerfinance.github.io/gamechanger.wallet/docs/gcscript/syntax.html`
- ISL: `https://gamechangerfinance.github.io/gamechanger.wallet/docs/gcscript/ISL.html`
- Schema/reference URLs: [schema.md](schema.md#references)
- Encoder and callback examples: https://github.com/GameChangerFinance/gamechanger
- Wallet response decoding: https://gamechangerfinance.github.io/gamechanger.wallet/docs/transactions/quick-start.html
