# GameChanger Build and Validation Workflow

This reference is based on the official `GameChangerFinance/gamechanger` library/CLI build features.

For schema flavors, directive transformations, validation, examples and diagnostics, read [schema.md](schema.md). This file covers installed build tools and resource policy.

For a single final JSON intent or a target without build tooling, use [dependency-free.md](dependency-free.md). Building a source project with this API is optional, not a prerequisite for UDC integration.

## Project resources and targets

- A project can contain GCScript, JSON/JSONC, documentation, binary assets and imported data. Preserve a general file map; apply GCScript processing only to intended executable sources.
- Supply the GC builder with the complete relevant virtual filesystem, including unsaved content, using its documented file-entry format. Pass selected source text as `input` and its absolute logical URI as `fileUri` (for example `app:///scripts/main.gcscript`). `fileUri` identifies the parent for relative imports; it does not read or select the input text.
- Make the target explicit. A project build can use a configured entrypoint; a file-run command can use the active executable file. These are application policies, not language requirements. Display the chosen target and build that target's dependencies before encoding its output. Never run an unrelated stale build artifact.
- Prefer the GC build machinery for imports. If unavailable, follow the transformations in [schema.md](schema.md#build-stage-directives); do not introduce incompatible scope or conversion rules.
- For project-backed result capture, distinguish a Project from a wallet Workspace. Keep the project root separate from files; reject empty/invalid destination paths at write boundaries. Bind asynchronous editor/result writes to project identity and revision, and cancel or reject stale callbacks after project close/switch.
- Before adding Buffer, crypto or CDN packages, inspect the exact GC browser bundle/version and documented exports (including `gc.utils.Buffer` where available). Feature-test browser APIs separately; do not assume Node globals or that library utilities provide browser-only APIs.

## Import URI Rules

- Imports must be protocol-qualified. Protocol-less paths like `./common.gcscript.jsonc` are rejected.
- Prefer `app://./common.gcscript.jsonc` for sibling imports and `app:///lib/common.gcscript.jsonc` for app-root imports.
- Default allowed protocol is `app`.
- `file://` is disabled by default and should be enabled only for trusted local builds.
- `http://` and `https://` are mutable remote resources; restrict with allowed domains when enabling.
- Mutable remote resources cannot safely import nested local resources.

## CLI Patterns

Build a JSONC project source:

```bash
gamechanger-cli build -f ./src/intents/open.gcscript.jsonc --cwd ./src/intents -o ./dist/intents/open.gcscript.json
```

Generate a snippet/dapp:

```bash
gamechanger-cli preprod snippet html -v 2 -S -o ./dist/index.html -f ./dist/intents/open.gcscript.json --snippetArgsFile ./config/snippet-args.json
```

## Library Patterns

```javascript
import gc from '@gamechanger-finance/gc'

const useSchema = await gc.utils.downloadGCScriptSchema()
const outputDataURI = await gc.build.file({
  input: sourceJSONC,
  fileUri: 'app:///main.gcscript',
  files,
  useSchema,
  compactOutput: true
})
const built = gc.utils.dataURIToBuffer(outputDataURI).toString('utf8')

```

## Generate a dapp from an intent

Before writing the intent, use [schema.md](schema.md) for valid function inputs, ISL and return-aware dataflow. Find a relevant starting point in the [official example catalog](https://github.com/GameChangerFinance/gamechanger.wallet/blob/main/examples/README.md); load only the chosen example.

Choose `html` or `react` for the rich templates with argument forms; `html-zero` provides a minimal native-API frontend without the rich form generator. Generated rich templates can load browser libraries even though generating a file does not require a frontend build system. For no-library targets, use `html-zero` or [dependency-free.md](dependency-free.md).

### Arguments that become form fields

Put editable defaults in the root script's **`args` object**. Rich templates infer controls from these values: string → text, boolean → checkbox, number → numeric input, array/object → JSON editor. An empty/missing `args` object gives no inferred argument fields. Fields are derived from the supplied keys/default values, not automatically from the complete language schema. Keep blockchain quantities as strings.

```json
{
  "type": "script",
  "title": "Send a message",
  "description": "Return the message entered in this app.",
  "args": { "message": "Hello Cardano" },
  "exportAs": "message",
  "return": { "mode": "last" },
  "run": { "message": { "type": "macro", "run": "{get('args.message')}" } }
}
```

The generated form edits `args.message`; the macro consumes that edited value. Defaults alone do not connect fields to execution: read them with ISL and proxy args into nested scripts where needed. `title` and `description` supply intent context. UI labels/control overrides belong in the generated template's field configuration, not invented GCScript argument descriptors. `snippetArgs` customizes generator tokens; it is distinct from runtime `args`. Its overrides are injected as source text, so use trusted values and valid escaped literals for the documented token context.

### Library or CLI

With the GC library already available, pass the built strict JSON intent as `input`:

```javascript
const dataURI = await gc.snippet.html({
  input: JSON.stringify(intent),
  apiVersion: '2', network: 'preprod', encoding: 'gzip'
});
const html = gc.utils.dataURIToBuffer(dataURI).toString('utf8');
```

Save `html` as `index.html`, or offer it as a browser download. Select `gc.snippet.react` or `gc.snippet['html-zero']` for the other formats. The CLI command above generates the same format family; `--snippetArgsFile` is needed only for custom template overrides. Multi-intent `snippetArgs.defaultIntents` overrides replace the default intent map; inspect the library documentation before using that advanced option.

### Playground: no local installation

Open [Playground](https://wallet.gamechanger.finance/playground), load/edit the intent, select the intended network/encoding and **Generate link**. In **Dapp Creator**, choose **Generate HTML + JS**, **Generate HTML + React**, or **Generate HTML Zero (small footprint, no dependencies)**. Download the generated source, inspect/customize it, and serve or deploy it. Use the rich formats for inferred argument forms. Verify the generated artifact with your deployed library/template version; UI wording can evolve.

### Host-adaptive return URL

Current rich templates inject a clean callback before encoding intents with `exportAs` or `return`; HTML Zero also derives its callback from the served page. For a custom app, inject the callback **at runtime on its deployed host**, before packing the script:

```javascript
const current = new URL(window.location.href);
const code = JSON.parse(JSON.stringify(intent));
code.returnURLPattern = current.origin + current.pathname + current.hash;
// Encode code, not the original intent, using GC or the native UDC helpers.
```

This preserves the host, port, deployment subpath and hash while omitting stale result queries. If routing requires query state, deliberately preserve the needed parameters and remove the consumed payload. Do not use only `hostname` or hardcode the generation machine's origin. Use a callback page the host actually serves. After return, decode/persist exports and clean history as described in [the return lifecycle](debugging-devex.md#complete-wallet-return-lifecycle).

## Validation

Use the commands, contracts and report guidance in [schema.md](schema.md#validation-and-diagnostics). Build once and validate the resolved strict JSON; do not repeat a long build merely to obtain another validation report.

## Sources

- GC build API and virtual files: https://github.com/GameChangerFinance/gamechanger#build-multi-file-gcscript-projects
- Snippet API and overrides: https://github.com/GameChangerFinance/gamechanger#snippet-customization-html--html-zero--react--button--express
- Rich HTML form/callback behavior: https://github.com/GameChangerFinance/gamechanger/blob/master/src/handlers/snippet/templates/template-html.ts
- Rich React template: https://github.com/GameChangerFinance/gamechanger/blob/master/src/handlers/snippet/templates/template-react.ts
