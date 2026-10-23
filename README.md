[![Hits](https://hits.sh/github.com/GameChangerFinance/gcscript-dsl-skill.svg)](https://hits.sh/github.com/GameChangerFinance/gcscript-dsl-skill/)

# GCScript DSL skill

Agent instructions for writing and validating GCScript/ISL, assembling multi-file projects, and integrating wallet URL/QR intents and returned results.

The skill follows the [Agent Skills format](https://agentskills.io/specification). Its entry point is [skills/gcscript-dsl/SKILL.md](skills/gcscript-dsl/SKILL.md); supporting references are loaded only as needed. Reading the skill and generating code require no package installation. 

## Install with a skill manager

For tools supported by the [skills CLI](https://github.com/vercel-labs/skills), run:

```sh
pnpm dlx skills add GameChangerFinance/gcscript-dsl-skill --skill gcscript-dsl
```

Select your target agent when prompted. The installer handles supported agent-specific locations; consult its documentation for prerequisites and options. This command installs instructions, not the GC library or CLI.

## Manual installation

Download or clone this repository and copy the `gcscript-dsl/` directory from `skills/` into your tool's documented skill directory. Preserve `SKILL.md`, `references/` and their relative paths. The optional `agents/openai.yaml` provides client UI metadata; other consumers can use the core Markdown files directly.

For tools accepting skill archives, run the packaging command below. The ZIP contains `gcscript-dsl/SKILL.md` and its supporting files. Follow the target tool's upload requirements; ZIP support and accepted layouts vary.

## Use in a web chat

If the chat has no skill loader, attach or provide the contents of `SKILL.md`, then supply only the reference needed for the task. A repository URL alone does not guarantee the assistant can retrieve linked files. Provide downloaded official schemas when its execution environment cannot fetch them.

| Task | Reference |
|---|---|
| Dependency-free browser/Python integrations and constrained hardware | [dependency-free.md](skills/gcscript-dsl/references/dependency-free.md) |
| Schema discovery, return-aware code generation, scopes and validation | [schema.md](skills/gcscript-dsl/references/schema.md) |
| Multi-file builds and resource resolution | [build-validate.md](skills/gcscript-dsl/references/build-validate.md) |
| Wallet encoder options, URLs, QR and callback contracts | [gcscript-udc.md](skills/gcscript-dsl/references/gcscript-udc.md) |
| Result capture, popup/redirect/QR flows and troubleshooting | [debugging-devex.md](skills/gcscript-dsl/references/debugging-devex.md) |

Example request:

> Use this skill to write a GCScript script that retrieves and exports my payment key hash. Inspect the official input and return contracts, explain the required permission, and distinguish schema review from automated validation. Do not launch a wallet for execution yet.

For tool integrations supporting named skills, invoke `$gcscript-dsl` or your tool's equivalent. Instructions do not authorize signing, submitting transactions or publishing data.

## Source of truth

Use the [official wallet language reference](https://wallet.gamechanger.finance/doc/api/v2/) and [GC library documentation](https://github.com/GameChangerFinance/gamechanger) for current behavior. Official schema URLs and selective retrieval examples are in [schema.md](skills/gcscript-dsl/references/schema.md#references).

## Package a skill ZIP

Download the ready-to-import [gcscript-dsl.zip](gcscript-dsl.zip), or rebuild it after editing the skill using the instructions below. Keep the archive synchronized with the skill sources when publishing.

The skill contains only Markdown instructions, license text and optional YAML metadata. No packaging script or executable source is included.

From the repository root, create the ZIP with your operating system's archive tool, or run:

```sh
cd skills
zip -r ../gcscript-dsl.zip gcscript-dsl
```

The archive must contain exactly one top-level `gcscript-dsl/` folder, with `SKILL.md` directly inside it. Include its `references/`, `agents/` and `LICENSE`; exclude `.git/`, the repository README and unrelated files. Do not ZIP the entire repository or add an extra parent folder. Check the target tool's upload requirements before importing.

## Discovery and listings

The canonical package is `skills/gcscript-dsl/`, matching the [skills CLI discovery layout](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md). It has standard YAML frontmatter, relative references and optional client metadata; no tool-specific execution permissions are required.

To inspect discoverable skills without installing them, use a trusted installed skills CLI, or:

```sh
pnpm dlx skills add GameChangerFinance/gcscript-dsl-skill --list
```

This queries the published repository. [skills.sh](https://skills.sh/docs) describes installation-based discovery/ranking; other directories may require separate submissions and their own review. Check the target directory's current requirements before submitting. No listing submission is required to download and use this skill.

## License

[MIT](LICENSE). The same license is bundled inside the skill for directory and ZIP consumers.

Created by zxpectre, at GameChangerFinance.
