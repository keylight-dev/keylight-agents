# Keylight for Agents

Two skills for adding Keylight licensing and debugging an existing integration:

- `adding-keylight` detects the app stack, configures the SDK and signing keys,
  wires activation and entitlement checks, and verifies the integration.
- `debugging-licensing` traces a failure from the API response through device,
  lease and local verification, then checks the correction.

The skills work without the Keylight CLI or a connected account. An authenticated
CLI can add account context and perform authorized setup; it is optional.

The public source lives in [keylight-dev/keylight-agents](https://github.com/keylight-dev/keylight-agents),
alongside the SDK repositories. This is an early 0.1.0 draft; it has no marketplace
listing yet. The same `skills/` source serves each agent host; packaging files
contain no licensing rules.

## Try locally

Clone the package to a directory of your choice:

```bash
git clone https://github.com/keylight-dev/keylight-agents.git
```

In Claude Code, start a session in the application you want to license and point
at this checkout:

```bash
claude --plugin-dir /absolute/path/to/keylight-agents
```

Use `/keylight:adding-keylight` or `/keylight:debugging-licensing`, or ask naturally:

> Add Keylight licensing to this Tauri React app. Keep verification in Rust.

> Activation says “lease signature verification failed.” Find the cause and fix it.

Claude Code uses `.claude-plugin/plugin.json` and the shared `skills/` directory.
See the [official plugin guide](https://code.claude.com/docs/en/plugins).

For Codex, copy the two skill directories from this checkout into the target
application's `.agents/skills/` directory, preserving any existing skills. For
user-wide installation, the supported directory is `~/.agents/skills/`. Open a
new conversation and use `$adding-keylight` or `$debugging-licensing`.
See [OpenAI's skill installation locations](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills).

The root `plugin.json` also follows the
[portable Agent Plugins format](https://developers.openai.com/plugins/build/plugins).
Directory discovery is the documented local Codex installation path for this draft;
installation in a fresh agent conversation and marketplace distribution have not
been tested. Other agent hosts can consume the same SKILL.md files, but their
installation adapters are not included yet.

## Maintain and verify

Keep SDK methods, release requirements and API behavior in the canonical
[Keylight docs](https://docs.keylight.dev/) and SDK repositories. Update a skill
only when it changes how the agent should integrate or debug the product.

Validate both SKILL.md files with a standard skill validator and the Claude
manifest with `claude plugin validate /absolute/path/to/keylight-agents`.
The [behavioral scenarios](tests/scenarios.md) cover the first workflows to try
before publishing. Syntax validation does not establish that an agent completed
those workflows successfully.
