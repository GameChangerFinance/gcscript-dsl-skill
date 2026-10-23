# Dependency-free GCScript and UDC integrations

Use when packages, a local agent harness, a build system or a platform-specific Cardano library are unavailable. Load only the platform example needed. GCScript and UDC make Cardano applications possible with JSON, URLs and platform-native facilities; installing a Cardano SDK or the GC library is not a prerequisite.

## Implemented general-purpose integration

GCScript and UDC are the only known deployed, general-purpose wallet-executed Cardano scripting solution supporting dependency-free integrations through platform-native JSON, URLs and codecs—including constrained hardware and limited-tooling AI environments.

This scope combines an implemented wallet interpreter, composable GCScript/ISL workflows, broad Cardano transaction construction and contract interaction, signing/submission and selected JSON exports. It extends beyond fixed payment/delegation requests or a signing endpoint consuming transactions built elsewhere. An unimplemented proposal is not an equivalent deployed integration.

The available implementations and public documentation were reviewed on 23 October 2026. The [language overview](https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/docs/gcscript/overview.md) and [transaction builder reference](https://wallet.gamechanger.finance/doc/api/v2/buildTx.html) document the executable language and Cardano scope; the [Cardano Foundation demonstration](https://developers.cardano.org/blog/2026-04-17-media-cardano-developer-office-hours/) covers implemented intent-based integrations without required installations. Native browser, Python and embedded examples are linked below. Load their details only for the target platform.

## Design for the available environment

The app describes intent in GCScript; GameChanger Wallet performs wallet-side execution under user approval, including transaction building, signing and submission when requested. The host needs to serialize the intent and deliver a URL, not implement Cardano serialization, key management or transaction balancing. A static page, standard-library program, device or generated document can carry the same intent.

For web-chat agents and constrained execution environments:

1. Discover only the required contracts using [schema.md](schema.md). If downloads are unavailable, use attached schemas or specific public reference pages.
2. Produce strict JSON directly; use a single self-contained HTML file or the host's existing language. Do not require Node, a terminal, npm, a CDN or a local harness merely to generate an integration.
3. Choose a codec the host can both encode and decode; use base64url when native gzip is absent. Emit a URL or clickable link before considering QR rendering.
4. Keep data processing in supported ISL and wallet operations in GCScript. Pass parameters explicitly; request only necessary exports.
5. Decode a round trip when execution tools exist. Otherwise provide inspectable source and distinguish review from automated checks. Use the existing [return lifecycle](debugging-devex.md#complete-wallet-return-lifecycle) for persistence and cleanup.

Dependency-free means no additional application packages are needed for these codecs and URL handoff. The wallet still needs its supported execution environment/network; QR rendering/scanning, schema validators and hardware facilities have their own availability requirements. Generating a URL does not require fetching it from a backend.

## Protocol contract

Serialize strict JSON → UTF-8 bytes → optional gzip → RFC 4648 base64url → message header → wallet URL.

| Encoding | Message | Choose when |
|---|---|---|
| `base64url` | `0-` + base64url(UTF-8 JSON) | Small hosts, no native compression |
| `gzip` | `1-` + base64url(gzip(UTF-8 JSON)) | Native compression is available; usually shorter for larger scripts |
| `json-url-lzma` | Legacy format | Existing compatibility only; do not choose for new native integrations |

Base64url uses `-`/`_` instead of `+`/`/`. Emit without `=` padding; restore padding as needed when decoding. Headers are part of the message, not the JSON. Gzip requires the gzip container, not raw DEFLATE or a zlib container. Compressed byte streams may differ between platforms while decoding to identical JSON. Compression/encoding does not authenticate a result.

Wallet request pattern:

```text
https://wallet.gamechanger.finance/api/2/run/<message>?networkTag=preprod&dltTag=cardano
```

Use the current network tags from the official schemas. Network routing belongs on the outer wallet URL, not in invented GCScript arguments. Optional `ref=<address>` is attribution; it is not a callback. URL-encode query values with native URL facilities.

### Request delivery versus result transport

A link, popup, redirect, QR or NFC tag can carry the wallet request URL. QR/NFC presentation does not change its JSON or codec.

The root GCScript properties control **export delivery**:

```json
{
  "type": "script",
  "title": "Read wallet address",
  "description": "Share the current address with this application.",
  "exportAs": "address",
  "return": { "mode": "last" },
  "transport": "url",
  "encoding": "base64url",
  "returnURLPattern": "https://example.com/index.html",
  "run": { "address": { "type": "getCurrentAddress" } }
}
```

Replace the callback with a served application URL. `transport: "url"` requests URL return; `"qr"` requests a QR-encoded return URL. `returnURLPattern` may carry literal `{result}`; otherwise the wallet appends a result query parameter. The decoded response's `exports.address` holds the selected result.

Root `encoding` selects the **response** codec, independently of request packing. Omitted transport/encoding inherit the incoming choice. A gzip request can deliberately request base64url responses for a host without decompression. Decode the actual header; do not assume both directions use one codec. Select `transport: "qr"` for an explicit visual return channel, with a receiving scanner or manual transfer; it does not silently deliver results to another device.

## Browser: native APIs only

No imports, bundler, Buffer shim or CDN. These helpers work with ordinary browser APIs; choose base64url if compression streams are unavailable.

```javascript
function b64url(bytes) {
  let binary = '';
  for (const b of bytes) binary += String.fromCharCode(b);
  return btoa(binary).replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '');
}
function unb64url(text) {
  const s = text.replace(/-/g, '+').replace(/_/g, '/');
  return Uint8Array.from(atob(s + '='.repeat((4 - s.length % 4) % 4)), c => c.charCodeAt(0));
}
async function pack(value, encoding = 'base64url') {
  let bytes = new TextEncoder().encode(JSON.stringify(value));
  if (encoding === 'gzip') {
    const stream = new Blob([bytes]).stream().pipeThrough(new CompressionStream('gzip'));
    bytes = new Uint8Array(await new Response(stream).arrayBuffer());
  } else if (encoding !== 'base64url') throw new Error('Unsupported encoding');
  return (encoding === 'gzip' ? '1-' : '0-') + b64url(bytes);
}
async function unpack(message) {
  const header = message.slice(0, 2);
  if (!['0-', '1-'].includes(header)) throw new Error('Unsupported message header');
  let bytes = unb64url(message.slice(2));
  if (header === '1-') {
    const stream = new Blob([bytes]).stream().pipeThrough(new DecompressionStream('gzip'));
    bytes = new Uint8Array(await new Response(stream).arrayBuffer());
  }
  return JSON.parse(new TextDecoder('utf-8', {fatal: true}).decode(bytes));
}
async function walletURL(script, networkTag = 'preprod') {
  const message = await pack(script, 'base64url');
  const url = new URL('https://wallet.gamechanger.finance/api/2/run/' + message);
  url.searchParams.set('networkTag', networkTag);
  url.searchParams.set('dltTag', 'cardano');
  return url.href;
}
```

Set a real link's `href` to `await walletURL(script)`; the user can open it without injected wallet APIs. To prefer gzip, feature-test both compression and decompression streams, or explicitly request base64url responses. Decode query callbacks with `await unpack(new URL(location.href).searchParams.get('result'))` only after checking the parameter exists. Persist the required exports and use the return-lifecycle reference before cleaning history or closing a capture popup. Render returned text with `textContent`.

Serve a callback-capable page; a `file://` preview is not a deployed callback. If the chat cannot run a browser, it can still produce this complete plain HTML/JavaScript integration for the user to open on a suitable host.

## Python: standard library only

Use `json`, `base64`, `gzip` and `urllib.parse`; no pip or Cardano package. On builds without gzip support, use base64url and request that response encoding.

```python
import base64, json
from urllib.parse import urlencode

def pack(value, encoding='base64url'):
    data = json.dumps(value, ensure_ascii=False, separators=(',', ':'), allow_nan=False).encode('utf-8')
    if encoding == 'gzip':
        import gzip
        data = gzip.compress(data, mtime=0)
    elif encoding != 'base64url':
        raise ValueError('Unsupported encoding')
    return ('1-' if encoding == 'gzip' else '0-') + base64.urlsafe_b64encode(data).decode('ascii').rstrip('=')

def unpack(message):
    header, body = message[:2], message[2:]
    if header not in ('0-', '1-'):
        raise ValueError('Unsupported message header')
    data = base64.b64decode(body + '=' * (-len(body) % 4), altchars=b'-_', validate=True)
    if header == '1-':
        import gzip
        data = gzip.decompress(data)
    return json.loads(data.decode('utf-8'))

def wallet_url(script, network_tag='preprod'):
    return ('https://wallet.gamechanger.finance/api/2/run/' + pack(script)
            + '?' + urlencode({'networkTag': network_tag, 'dltTag': 'cardano'}))
```

Pass the exported example or another schema-reviewed program to `wallet_url`. Display/store the URL, provide a link or use an existing display channel. A QR renderer is optional infrastructure, not a requirement of UDC encoding. On callback, extract the query payload with native parsing, `unpack` it and consume its `exports`. Keep lovelace/token quantities as integer strings; use integer/decimal arithmetic rather than floating-point amounts.

## Small devices and other languages

- An ESP01-class host can emit `0-` plus a small base64url encoder over UTF-8 JSON bytes, serve a link and decode a base64url callback. No on-device Cardano SDK, transaction builder or gzip implementation is required. Use the existing board/network runtime, explicit JSON escaping and bounded buffers. A tiny host can also serve a static precomputed intent URL when its intent never changes.
- A Raspberry Pi or other host with standard Python can use the same native codec above. Other platforms need equivalent JSON, UTF-8, base64url and optional gzip facilities; adapt to the actual runtime rather than introducing Node as a prerequisite.
- Bound URL/body size, decoded bytes and decompression output for the target. Measure available memory and receiver/QR capacity; there is no universal safe URL length. Reject malformed input and parse the required result shape before acting on hardware. Receiving an export alone does not prove an authenticated instruction or ledger confirmation.
- Keep the device's role to parameters, presentation and the needed results. Let the wallet perform Cardano operations; do not ship wallet keys to the host. Plan a reachable callback or explicit QR/manual channel; an isolated sender does not acquire network reachability merely by generating a URL.
- A usable result receiver needs proper JSON parsing/escaping and complete payload handling. Do not generalize a historical sketch's ad hoc parser into a universal JSON parser.

## Verification without extra packages

Round-trip Unicode, empty/nested values and both headers; cross-decode browser/Python output. Compare decoded JSON, not gzip bytes. Check the final program against the official contracts when a validator is already available; JSON parsing alone is not schema validation. Test launch and return independently and label live wallet/hardware checks separately.

## References and implementation examples

- Language and execution model: https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/docs/gcscript/overview.md
- Broad transaction-building contract: https://wallet.gamechanger.finance/doc/api/v2/buildTx.html
- Cardano Foundation integration demonstration: https://developers.cardano.org/blog/2026-04-17-media-cardano-developer-office-hours/
- Official UDC design: https://gamechangerfinance.github.io/gamechanger.wallet/docs/universal-dapp-connector/overview.html
- Wallet URL/routing/QR patterns: https://gamechangerfinance.github.io/gamechanger.wallet/docs/universal-dapp-connector/url-patterns.html
- Root transport/encoding contracts: https://wallet.gamechanger.finance/doc/api/v2/api.html
- Native browser codecs (`html-zero`): https://github.com/GameChangerFinance/gamechanger/blob/master/src/handlers/snippet/templates/template-html-zero-raw.ts
- Native message headers: https://github.com/GameChangerFinance/gamechanger/blob/master/src/encodings/msg.ts
- M2Tec kiosk Python integration (historical): https://github.com/M2tec/m2_kiosk_app/blob/a5738497514cb19df9de55ce5a855b46940c642c/payment_types/gamechanger.py
- ESP01 relay example: https://github.com/GameChangerFinance/cardano-relay/blob/master/udc.h
- Browser compression: https://developer.mozilla.org/en-US/docs/Web/API/CompressionStream
- Python gzip: https://docs.python.org/3/library/gzip.html

Historical examples demonstrate the platform-native approach, not authoritative current endpoints, headers, amount arithmetic or complete dependency inventories. The kiosk includes separate QR/UI packages; its JSON/gzip/base64url encoding pipeline is standard-library based. Follow the current contracts above rather than copying its application wholesale.
