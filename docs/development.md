# Development guide

For installation, invocation and updates, see the [README](../README.md).
This guide covers package structure, local testing, validation and distribution.

## Package

```text
plugin.json                     Portable Agent Plugins manifest
skills/atdd-workflow/SKILL.md    Shared workflow
com.github.copilot/commands/atdd.md  Copilot slash-command entry point
.agents/plugins/marketplace.json  Codex distribution catalogue
.claude-plugin/plugin.json       Claude Code compatibility manifest
.claude-plugin/marketplace.json  Claude Code distribution catalogue
```

The skill contains all workflow instructions. The manifest provides packaging
metadata only. There are no runtime dependencies, hooks, MCP servers or custom
test frameworks. Use the target project's existing test tools.

The package follows the [Agent Plugins 1.0.0 specification](https://agent-plugins.org/specification)
and its [manifest schema](https://agent-plugins.org/schemas/1.0.0/plugin.schema.json).
Skills are discovered from the fixed `skills/` directory. A small Copilot
command lives in the standard's `com.github.copilot` extension namespace;
it loads the shared skill without duplicating the workflow. Other clients
can ignore this namespace and use the skill directly. Claude Code's
compatibility manifest and catalogue sit in `.claude-plugin/` and load the
same skill. Root `plugin.json` remains the portable manifest. See the
[client extension rules](https://agent-plugins.org/plugin-authors/client-extensions).

The standard defines packaging and discovery; installation and invocation are
client-specific. Choose a [compatible client](https://agent-plugins.org/compatible-clients)
for full plugin testing. Standalone skill support alone does not establish
support for the plugin format.

## Local testing

### Claude Code

From a disposable target project, run:

```sh
claude --plugin-dir /absolute/path/to/agent-plugin-atdd
```

Invoke `/atdd:atdd-workflow` with a small feature request. The namespaced
skill is the command; no initialisation prompt is required. This integration
targets Claude Code; Claude chat and Cowork installation are unverified.
See [Claude Code plugins](https://code.claude.com/docs/en/plugins).

### Codex

The repository's `.agents/plugins/marketplace.json` points to the portable
package at the repository root. Its `source: local` means local to the
fetched Git snapshot; consumers do not need a local checkout.

Codex installs local plugins through a marketplace catalogue. Keep this
host-specific test catalogue outside the portable package. From this
repository's root, create a temporary marketplace containing a package copy:

```sh
python3 - <<'PY'
import json
import shutil
import tempfile
from pathlib import Path

root = Path(tempfile.mkdtemp(prefix="atdd-local-"))
plugin = root / "plugins" / "atdd"
plugin.mkdir(parents=True)
for name in ("plugin.json", "README.md", "LICENSE"):
    shutil.copy2(name, plugin / name)
shutil.copytree("docs", plugin / "docs")
shutil.copytree("skills", plugin / "skills")
shutil.copytree("com.github.copilot", plugin / "com.github.copilot")
shutil.copytree(".claude-plugin", plugin / ".claude-plugin")
catalogue = root / ".agents" / "plugins" / "marketplace.json"
catalogue.parent.mkdir(parents=True)
catalogue.write_text(json.dumps({
    "name": "atdd-local",
    "interface": {"displayName": "ATDD Local"},
    "plugins": [{
        "name": "atdd",
        "source": {"source": "local", "path": "./plugins/atdd"},
        "policy": {"installation": "AVAILABLE", "authentication": "ON_INSTALL"},
        "category": "Productivity"
    }]
}, indent=2) + "\n")
print(f"codex plugin marketplace add {root}")
print("codex plugin add atdd@atdd-local")
PY
```

Run the two commands printed by the script. Open a fresh Codex session in a
small project with an existing test runner, then request:

```text
Use $atdd-workflow for story US-003 in docs/user-stories.md.
```

In the desktop app, restart the app, open the Plugins Directory, choose
**ATDD Local** and install or enable **atdd**. The host installs a cached copy;
edits to this checkout do not automatically update it. To retest edits, remove
the test install and marketplace, then recreate and install the package:

```sh
codex plugin remove atdd@atdd-local
codex plugin marketplace remove atdd-local
```

You can delete the temporary directory after removing the marketplace.
See the [Codex local plugin guide](https://developers.openai.com/plugins/build/plugins).

### Quick workflow-only testing

Ask your agent to read this checkout's `skills/atdd-workflow/SKILL.md` by
absolute path and follow it for a small feature in the current project.
This tests the instructions without testing plugin discovery or installation.
Alternatively, copy the skill folder into the host's documented skills directory.

### VS Code / Copilot notes

The bundled `/atdd` command loads the shared skill. The skill can also be
invoked directly where the host exposes it. A diagnostic prompt asking the
agent to locate `SKILL.md` is not an initialisation step, and installing the
plugin does not add a separate agent to the agent selector.

For local testing, use **Chat: Install Plugin from Source** and select this
checkout. Verify that the command reads the bundled skill and follows the
workflow; command discovery alone does not prove execution. Availability
depends on the selected harness and organisation policy.

## Validation

Edit `skills/atdd-workflow/SKILL.md` to change the workflow. Validate
`plugin.json` against the canonical manifest schema linked above and the
skill against the [Agent Skills specification](https://agentskills.io/specification).
Host-native manifest validators do not establish shared-standard conformance.
Keep the portable and Claude manifest identity, version, description and
author metadata consistent. Validate Claude packaging with
`claude plugin validate .claude-plugin/plugin.json --strict` and
`claude plugin validate .claude-plugin/marketplace.json --strict`.

For a manual behaviour check in a disposable project, request a small feature.
In Copilot, verify that the **atdd** command loads the bundled skill before
starting the workflow. Command discovery alone does not prove execution.
In Claude Code, invoke **/atdd:atdd-workflow** and check the same workflow.
Confirm that the agent helps create missing requirements, records project
context, and waits for agreement on the selected story, criteria and scenarios.
It should demonstrate a meaningful test failure and wait again before
implementation. Also check a missing requirements file, a stale context pointing
to a missing file, alternative requirements locations, incomplete backlogs,
reuse of a saved custom source and a blocked test environment. Before executable
tests begin, confirm that the agreed story, criteria and scenarios were saved
and read back from the chosen source, and the context records its actual status.

## Updates and distribution

This plugin is Markdown and JSON; there is no build or compilation step.
For quick editing cycles, ask the agent to read the current skill by absolute
path in a fresh session. For installed-plugin tests, refresh the package copy
and installation as described above, then start a fresh session.

For an initial public release, a Git repository with versioned releases and a
host-supported marketplace entry is sufficient. The portable plugin format
does not prescribe a registry. Codex supports Git-backed marketplace sources
and optionally npm-backed plugin entries; npm publication is not required.
Keep host-specific distribution metadata separate from the portable package.
Publish new versions rather than silently replacing a released version.
Refreshing a Git marketplace updates its snapshot; confirm the installed
plugin version separately. Marketplace distribution is separate from listing
in OpenAI's public plugin directory, which has its own submission process.
See the [Codex packaging and distribution guide](https://developers.openai.com/plugins/build/plugins).
