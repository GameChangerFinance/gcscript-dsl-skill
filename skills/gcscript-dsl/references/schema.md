# GCScript schemas for agents and tools

Use for schema discovery, schema-driven coding, permissions, return contracts and validation. No installation is required to inspect schemas or generate code. Load only the section needed.

## Choose a flavor and retrieval path

| Task | Flavor |
|---|---|
| Learn functions, arguments, results and examples; generate clients or visual blocks | Full |
| Validate wallet-facing JSON with less download/storage overhead | Minimal |
| Validate while retaining examples and descriptive diagnostics | Full |

Both retain reusable common types. Full additionally includes GCScript return contracts, ISL argument/return contracts and examples.

- Browsing only: open the reference index, then selected function/type pages.
- Python/JavaScript execution: load a downloaded/attached bundle and print selected nodes.
- No execution-tool network: use an attachment; browsing and execution may have separate files/network.
- No validator available: perform a schema-guided review; do not claim automated validation.

Download once per task and keep the parsed bundle outside conversational context. Discover names first, fetch selected contracts second, then follow only relevant references. Print one applicable example, not every example. Retain document/pointer pairs as retrieval handles; track visited pairs to avoid recursive expansion. Record retrieval date for reproducibility.

## Schema download budget

Measured serialized bundle sizes on 23 October 2026 (decimal kB; 1 kB = 1,000 bytes):

| Flavor | Bytes | kB |
|---|---:|---:|
| Full | 1,636,706 | 1,636.706 |
| Minimal | 542,979 | 542.979 |

These are file sizes, not token counts, compressed transfer sizes or parsed-memory requirements. Both can exceed a small context budget. Download/parse outside context if possible, extract selected nodes, and remeasure when fetching a newer snapshot. Use full for return-aware generation and examples; minimal for input validation, not as a substitute for missing result/ISL contracts.

## Compact code for QR, NFC and small devices

Prefer the smallest semantically equivalent GCScript when possible. Optimize the delivered intent, not merely the appearance of source code. Preserve required arguments, safety assertions, types, scope, ordering, permissions and intended exports.

1. Keep readable authoring source; serialize strict JSON without indentation before packing. Measure both gzip and base64url when supported: smaller JSON is not always smaller after compression. Reuse identical strings for repeated meanings where practical so gzip can exploit repeated byte sequences; avoid unnecessary spelling or wording variations, and measure the packed result.
2. Merge independent assertion calls into one `assert(and(...),'Concise message')` when the checks can all be evaluated safely. Do not assume boolean helpers short-circuit: retain separate guards when later expressions require a validated type/path. Remove duplicated checks only when the same invariant is already established for the same unchanged value; never weaken safety to meet a byte target.
3. Shorten repeated descriptions, errors and nonessential titles; retain useful user consent context. Omit only documented defaults and genuinely unused data. Shorten user-defined step/argument names consistently with all ISL paths; never rename language properties.
4. Use a `macro` to construct a result object/array directly when it replaces several pure value-building child functions. A `script` performs function execution, scopes and return/export selection: do not replace it when those semantics or wallet operations are needed.
5. For byte-critical internal sequences, arrays can avoid repeated map keys where both contracts and consumers support positional data. Update indexes and preserve execution order/result shape. Do not replace required keyed objects, break public return contracts or bypass dictionary semantics merely to save bytes.
6. Reuse existing computed values rather than repeating equivalent calls; pass only necessary args and export only the required fields. Use stable public imports when appropriate, accounting for resource availability/trust and wrapper overhead; keep direct self-contained intents when that is the intended delivery model.
7. Measure the complete outbound and return URLs after headers, encoding, routing parameters, attribution and callback patterns. Shortening JSON alone does not prove NFC/QR compatibility. If it still does not fit, choose a larger tag, supported alternate transport or deliberately hosted resource flow rather than silently removing behavior.

### QR byte budgets

GC encodes the complete wallet URL into a QR. Its underlying byte-capacity table reaches the following Version 40 ceilings:

| Error correction | Maximum byte-mode content |
|---|---:|
| L | 2,953 bytes |
| M | 2,331 bytes |
| Q (current GC style default) | 1,663 bytes |
| H | 1,273 bytes |

These are URL-content ceilings, not raw JSON budgets or PNG/SVG file sizes. The lower-case URL/base64url payload uses byte-mode budgeting; do not use numeric/alphanumeric-mode capacities. Actual QR version, renderer settings, image size, logos, print quality and receiving camera affect usability. Leave margin and test real scans. Do not reduce error correction indiscriminately to force an oversized payload to fit. Increasing image pixels does not increase the Version 40 data ceiling.

### NFC memory budgets

Common tag user-memory capacities: NTAG213 **144 bytes**, NTAG215 **504 bytes**, NTAG216 **888 bytes**. Smaller tags can offer only **48/128 bytes**; NTAG I²C plus 2K offers **1,912 bytes**. User memory is not the maximum plain URL length: account for URI-prefix compression, NDEF record headers, URI prefix byte, record-length changes and tag/container overhead. Actual formatted free space and writer behavior determine fit. A URL fitting a QR can still exceed most small NFC tags.

Use UTF-8 byte counts (`new TextEncoder().encode(url).length` or `len(url.encode('utf-8'))`) for URL measurement, then the actual NDEF writer/tag capacity for NFC. Do not equate JavaScript string length with bytes or advertise a fit based solely on raw EEPROM size.

Capacity references: [NXP tag specifications](https://www.nxp.com/products/NTAG213_215_216), [QR versions](https://www.qrcode.com/en/about/version.html), [GC QR encoding](https://github.com/GameChangerFinance/gamechanger/blob/master/src/encodings/qr.ts), [GC QR styles](https://github.com/GameChangerFinance/gamechanger/blob/master/src/config/styles.ts).

## Bundle map and search paths

A bundle is a map of schema documents, not a single root schema.

| Lookup | Purpose |
|---|---|
| `api.json` | Complete wallet-facing program |
| `script.json` | Script/block input contract |
| `<function>.json` | Function input contract |
| `<function>.json#/definitions/return` | Result reference; full only |
| `returnTypes.json#/definitions/<function>` | Result contract; full only |
| `common.json#/definitions/<Type>` | Shared input/output type |
| `lang.json` | ISL syntax and grouped declarations |
| `permissions.json#/properties/<permission>` | Permission description/metadata |
| `<function>.json#/definitions/__api__/properties` | Execution/permission metadata |

Discover executable function documents by `definitions.__api__`; use `properties.type.const` as their invocation discriminator. Definition documents are not executable functions.

Inspect `required`, `properties`, `additionalProperties`, `$ref`, `const`, `enum`, bounds and composition. Optional does not imply nullable. `oneOf` requires exactly one matching branch; `anyOf` requires at least one.

### ISL support is explicit

Follow the relevant argument contract, including referenced schemas and composition branches. **If it does not include a reference to `lang.json`, ISL is not supported there.** A generic string or unrestricted JSON value does not imply evaluation.

`data.value` holds literal data: an ISL-looking string stays data. Use `macro.run` to evaluate ISL and produce values. Do not recursively evaluate arbitrary JSON strings.

## Contextual language tooling

GCScript is a DSL serialized as JSON. Parse JSON/JSONC syntax separately from language semantics; classify nodes by their resolved schema location and function discriminator.

- Names under `run`, metadata maps and other dictionaries can be user-defined. Names such as `build`, `sign`, `payment` or numeric labels are not reserved merely because they occur elsewhere in the language. Distinguish declared `properties`, `patternProperties` and `additionalProperties` at the actual location.
- Recognize ISL only at schema-supported positions and with valid expression syntax. Do not highlight function-like words in ordinary strings or keys as executable ISL; permit evaluated keys only where explicitly documented.
- Read ISL signatures from `definitions.arguments` and `definitions.return`. If contracts are unavailable, consult the corresponding reference; do not invent arity or result constraints.
- Separate lexical token coloring from schema-based semantic classification. Cache schema lookups by bundle identity, document/pointer and context; invalidate on schema/source changes.
- Documentation rendering must bound recursion depth, node count and output length, detect cycles and link large referenced definitions instead of expanding them. Treat `api.json`, `script.json` and `lang.json` as link-first display targets, not forbidden validation dependencies.
- Keep complete schemas registered for validation. Do not apply tooltip expansion limits or display exclusions to validators. Reuse the validator instance and compiled root per schema snapshot/options; avoid repeated compilation, cloning or full dereferencing during edits. Debounce checks and discard stale results. Minimal size does not guarantee faster compilation: measure compilation separately from repeated validation.

## Dependency-free extraction

Python standard library; use an attachment when network access is unavailable:

```python
import json
from urllib.request import urlopen
from urllib.parse import urldefrag, unquote

URL = 'https://wallet.gamechanger.finance/schema/api/v2/index.json.full'
B = json.load(urlopen(URL))
# Offline: B = json.load(open('index.json.full'))

functions = {n: s.get('title') for n, s in B.items()
             if '__api__' in s.get('definitions', {})}

def ref_node(ref, current):
    """Lookup bundle document names and JSON Pointer fragments."""
    doc, pointer = urldefrag(ref)
    doc = doc or current
    node = B[doc]
    pointer = unquote(pointer)
    if pointer:
        if not pointer.startswith('/'):
            raise ValueError('Expected JSON Pointer')
        for part in pointer[1:].split('/'):
            key = part.replace('~1', '/').replace('~0', '~')
            node = node[int(key)] if isinstance(node, list) else node[key]
    return doc, node

s = B['getCurrentAddress.json']
print(json.dumps({'required': s.get('required', []),
                  'arguments': list(s.get('properties', {})),
                  'return': s['definitions']['return'],
                  'example': s.get('examples', [])[:1]}))
```

The lookup helper is not a validator or general URI resolver. General tools must preserve base URIs and nested `$id` semantics. Do not fully inline the bundle: recursive script schemas can cycle and shared types multiply output.

Index ISL by function title, not hardcoded namespace positions:

```python
ISL = {f['title']: f
       for group in B['lang.json'].get('anyOf', [])
       for f in group.get('anyOf', [])
       if 'arguments' in f.get('definitions', {})}
print(json.dumps(ISL['getAddressInfo']['definitions']))
```

`definitions.arguments` describes positional argument arrays: tuple-valued `items`, `minItems`, `maxItems` and `additionalItems` describe arity; schema-valued `items` describes repeated arguments. `definitions.return` describes the JSON result. These aid tooling; actual ISL remains a code string.

## Runtime functions, scopes and memory

Executable GCScript nodes are function calls selected by `type`, including `script`, `macro` and `data`. Their arguments can contain ordinary data or, where supported, calls to ISL functions. An arbitrary object nested inside an argument is not automatically another executable GCScript node.

- `script.run` executes named child functions sequentially. Child results populate cache paths corresponding to their positions. A `macro` evaluates ISL in its `run` value; `data` returns literal `value`.
- `args` is the current call's supplied arguments. A script evaluates its own `args` and supplies that value to its children; `argsByKey` overrides it for matching child keys. Children without an override receive that script's `args`.
- A nested script does not implicitly forward its caller's args to its own children. Set `"args":"{get('args')}"` on that script to evaluate and proxy the incoming value before entering its children. Pass a smaller explicit object when only a few inputs are needed.
- `get('cache.step')` reads from the active cache base. Without isolation, nesting a script does not by itself rebase this lookup: include the nested path when needed.
- `isolateCache: true` on a script rebases its cache lookups to that script's subtree. Its children can use local names such as `cache.step`, regardless of where the script is composed. The previous cache base resumes after the script completes. Isolation changes cache addressing, not argument forwarding or global memory.
- Pass parent results through arguments when an isolated fragment needs them. Avoid adding isolation wrappers without adjusting cache paths. Cache isolation is a composition/scoping feature.
- `set('project.amount', value)` writes execution-wide user memory; read it with `get('global.project.amount')`. `global` is shared across nested scripts during the execution, including isolated ones; namespace keys to avoid collisions. It is not persistent wallet storage. Prefer explicit args/results for reusable fragments.
- A script's default result is a map of child results. `return.mode` selects its result (`last`, `one`, `some`, etc.); `none` returns undefined. The parent receives that result in its cache.
- `exportAs` on a script places its selected result under that key in exported results. Returning into a parent's cache and exporting to the dapp are separate choices.

For an isolated reusable child, use both `"isolateCache":true` for stable local cache names and `"args":"{get('args')}"` when its descendants need incoming arguments. One does not replace the other.

## Compose from input and return contracts

Select functions → read required inputs and relevant branches → follow shared types → inspect outputs → connect matching values → build → validate strict JSON.

### Export the user's payment key hash

Read the current address, decode it with ISL, and export only its payment key hash:

```json
{
  "type": "script",
  "title": "Payment key hash",
  "description": "Return the current address payment key hash.",
  "exportAs": "paymentKeyHash",
  "return": { "mode": "last" },
  "run": {
    "address": { "type": "getCurrentAddress" },
    "addressInfo": {
      "type": "macro",
      "run": "{getAddressInfo(get('cache.address'))}"
    },
    "paymentKeyHash": {
      "type": "macro",
      "run": "{get('cache.addressInfo.paymentKeyHash')}"
    }
  }
}
```

The contracts determine the connections: `getCurrentAddress` returns an address string, accepted by ISL `getAddressInfo(address)`. Its result is an address-info object, whose `paymentKeyHash` field the final macro selects. It is not a property of the original address string. A payment-script address has a script credential instead; its payment key hash may be empty. Check credential flags if a key credential is required.

The script's `last` result is the final macro's scalar; `exportAs` produces `exports.paymentKeyHash`. Configure UDC return transport separately when returning to a dapp; see [gcscript-udc.md](gcscript-udc.md).

For every connection, distinguish serialized transaction CBOR, transaction hashes and objects. Prefer shared type references for reusable sockets, then check structure and argument-selected variants. Do not assume optional fields or system-mode fields exist in ordinary results. Keep blockchain quantities in their documented BigNum representation and use ISL BigNum operations.

`common.json#/definitions/void` is `false`: no JSON value represents a result. Handle absent JavaScript results outside JSON; do not substitute null.

## Examples and permissions

Search the [official GCScript and dapp example catalog](https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/examples/README.md) for the closest workflow, then load only the relevant source files. Adapt their arguments and dataflow against the current input/return contracts; do not copy the whole catalog into context or assume every historical example matches current schema behavior.

Function `examples` contain invocations; return-schema `examples` contain JSON results. ISL call and return example arrays are separate, not index-paired. Read one example matching the chosen mode; constraints remain authoritative. Keep complete encoded values when showing real hashes/addresses/CBOR.

Metadata values live under schema `const` nodes:

```python
meta = B['getCurrentAddress.json']['definitions']['__api__']['properties']
values = {k: v['const'] for k, v in meta.items() if 'const' in v}
for name in values.get('permissions', []):
    print(B['permissions.json']['properties'][name])
```

Use permissions and their descriptions/severity to explain capabilities. Inspect runtime/preprocessor availability and custom permission-detection flags when applicable. A static permission union is a preview, not an authoritative prediction of dynamic behavior. Metadata does not grant permissions; wallet approval governs execution. Never insert metadata or return contracts into invocation arguments.

## Validation and diagnostics

Register bundle documents using their identities and `$id`; resolve references from that registry. Validate against `api.json` for complete programs, or `script.json` for reusable blocks, never against the outer map or `__api__`. Use a compatible JSON Schema validator supporting the published keywords and boolean schemas.

Validate the built strict JSON, not unresolved source directives. Input validation does not execute ISL, check evaluated argument/result contracts, prove cache paths, balance transactions or establish ledger acceptance. Tools may separately validate evaluated JSON against full result contracts, with absent results handled outside JSON.

Without a validator, parse strict JSON and review required fields, discriminators, branches, encodings and dataflow. State automated validation was not run; do not invent a partial validator.

With the GC CLI already available:

```sh
gamechanger-cli validate -f ./dist/main.gcscript.json -o ./dist/report.json
```

With the GC library already available, supply the chosen document map as `useSchema` to `gc.validate.file` or validated `gc.build.file`. Validation requires that schema unless intentionally disabled with `doValidate: false`. See [build-validate.md](build-validate.md) for build invocation and result decoding.

Reports provide validity, errors/warnings and locations. Treat invalid output as a blocker; inspect ISL warnings against the relevant declaration and accepted input field. For diagnostic-tool work, preserve instance paths and evidence, select the relevant composition branch and avoid hardcoded fixture fixes or exhaustive branch dumps. Prefer actionable suggestions and applicable examples.

## Build-stage directives

| Directive | Build output |
|---|---|
| `$importAsScript` | `script` wrapper; keyed `from` resources become `run` children |
| `$importAsData` | `data` wrapper; converted keyed resources become literal `value` |

`$importAsScript` preserves script options including `args`, `argsByKey`, `isolateCache`, `return` and `finally`. The build does not distribute `argsByKey` into imported children; normal script semantics handle it. `$importAsData` converts with `as: "object"`, `"json"`, `"string"`, `"hex"` or `"base64"`; other data properties pass through. Object parses JSON; json produces JSON text.

```json
{
  "type": "script",
  "run": {
    "settings": {
      "type": "$importAsData",
      "as": "object",
      "from": { "network": "app:///config/network.json" }
    },
    "connect": {
      "type": "$importAsScript",
      "args": "{get('args')}",
      "from": { "wallet": "app:///scripts/connect.gcscript" }
    }
  }
}
```

Use directives to share configuration and reusable fragments. JSONC sources can contain comments/trailing commas; emitted JSON is strict. Build directives resolve before wallet execution and differ from wallet `importAsScript`/`importAsData` functions. Use protocol-qualified app URIs; see [build-validate.md](build-validate.md) for resource policy and build commands.

Without a builder, generate final JSON directly or inline supplied files with the same wrappers, keyed maps and conversions. Resolve every nested resource; preserve scopes/args and validate the final artifact. Do not claim an official build when assembling manually.

## References

- Official example catalog: https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/examples/README.md

- Full: https://wallet.gamechanger.finance/schema/api/v2/index.json.full
- Minimal: https://wallet.gamechanger.finance/schema/api/v2/index.json.min
- Language: https://wallet.gamechanger.finance/doc/api/v2/
- Script/scopes: https://wallet.gamechanger.finance/doc/api/v2/script.html
- Macro: https://wallet.gamechanger.finance/doc/api/v2/macro.html
- Return types: https://wallet.gamechanger.finance/doc/api/v2/returnTypes.html
- Shared types: https://wallet.gamechanger.finance/doc/api/v2/common.html
- ISL: https://wallet.gamechanger.finance/doc/api/v2/lang.html
- Permissions: https://wallet.gamechanger.finance/doc/api/v2/permissions.html
- Official library/build: https://github.com/GameChangerFinance/gamechanger
- Validator reuse: https://ajv.js.org/guide/managing-schemas.html
- JSON Schema resolution: https://json-schema.org/understanding-json-schema/structuring
- Progressive disclosure/selective retrieval: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
