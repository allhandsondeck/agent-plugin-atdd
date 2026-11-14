# ATDD Agent Plugin

A small, shared agent skill for Acceptance Test-Driven Development (ATDD).
Develop software from agreed user stories and acceptance criteria using a
test-first ATDD workflow.

1. Locate requirements or help write the selected user story and its criteria.
2. Write Given / When / Then scenarios and wait for developer agreement.
3. Write acceptance tests, demonstrate that they fail for the intended reason,
   then pause for authorisation to implement.
4. Implement the agreed behaviour incrementally.
5. Verify, refactor and record the results.

## Requirements and project context

By default, the target project has two Markdown files:

```text
docs/user-stories.md    Product stories, acceptance criteria and scenarios
docs/atdd-context.md    Requirements location, stack, testing approach and setup
```

You can point the agent to another requirements file or directory. It records
that location and works directly with the source, avoiding duplicate documents.
If requirements are missing, the agent helps create them interactively.
The selected story must be sufficiently defined and agreed before executable
tests or implementation begin; other stories can remain incomplete.

A user story describes who needs what and why. Acceptance criteria define
observable conditions it must satisfy. Given / When / Then scenarios provide
concrete examples; acceptance tests verify those outcomes through code.
Keep story and criterion identifiers stable so tests can reference them.
Record test commands and evidence beneath the selected story or link existing
project evidence.

On first use, the agent inspects the project and asks only for missing
information. It reuses suitable test tools and agrees any new tooling before
introducing it. The testing boundary follows the behaviour, such as a browser
journey or API outcome. Later runs reuse and check the saved context. If the
test environment is blocked, the agent stops to resolve it with the developer.

## Package

```text
plugin.json                     Portable Agent Plugins manifest
skills/atdd-workflow/SKILL.md    Shared workflow
com.github.copilot/commands/atdd.md  Copilot slash-command entry point
```

The skill contains all workflow instructions. The manifest provides packaging
metadata only. There are no runtime dependencies, hooks, MCP servers or custom
test frameworks. Use the target project's existing test tools.

The package follows the [Agent Plugins 1.0.0 specification](https://agent-plugins.org/specification)
and its [manifest schema](https://agent-plugins.org/schemas/1.0.0/plugin.schema.json).
Skills are discovered from the fixed `skills/` directory. A small Copilot
command lives in the standard's `com.github.copilot` extension namespace;
it loads the shared skill without duplicating the workflow. Other clients
can ignore this namespace and use the skill directly. There are no
host-specific manifests. See the [client extension rules](https://agent-plugins.org/plugin-authors/client-extensions).

The standard defines packaging and discovery; installation and invocation are
client-specific. Choose a [compatible client](https://agent-plugins.org/compatible-clients)
for full plugin testing. Standalone skill support alone does not establish
support for the plugin format.

## Try it

### VS Code and the Agents window

In a VS Code version with agent plugin support enabled:

1. Open the Command Palette and run **Chat: Install Plugin from Source**.
2. Enter `https://github.com/allhandsondeck/agent-plugin-atdd.git`, or select
   this checkout's local folder.
3. Open **Chat: Open Customizations**, select **Plugins**, and confirm **atdd**
   is enabled for the intended harness and workspace. In the Agents window,
   open **Customizations** from the sidebar.
4. Start a fresh session and type `/` to find the **atdd** command (the host
   may show a plugin-qualified name). Select it and supply a user story or
   feature request. The command loads the shared **atdd-workflow** skill.

The command is the explicit Copilot entry point. The **atdd-workflow** skill
can also be invoked directly where the host exposes skills. A diagnostic
prompt asking the agent to locate `SKILL.md` is not an initialisation step.

If plugin support is disabled, enable `chat.plugins.enabled` in VS Code settings.
Availability depends on the selected harness and organisation policy. Installing
the plugin does not make it a separate agent in the agent selector.

To fetch published changes, open **Plugins** and select **Check for and Apply
Updates** (the refresh icon). No compilation or npm publication is needed.
See [agent plugins in VS Code](https://code.visualstudio.com/docs/agent-customization/agent-plugins).

### Full plugin testing with Codex

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
shutil.copytree("skills", plugin / "skills")
shutil.copytree("com.github.copilot", plugin / "com.github.copilot")
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

Claude Code is not currently on the Agent Plugins compatible clients list.
Its standalone skill support can be used for workflow testing; this package
does not include Claude's native plugin manifest.

## Development

Edit `skills/atdd-workflow/SKILL.md` to change the workflow. Validate
`plugin.json` against the canonical manifest schema linked above and the
skill against the [Agent Skills specification](https://agentskills.io/specification).
Host-native manifest validators do not establish shared-standard conformance.

For a manual behaviour check in a disposable project, request a small feature.
In Copilot, verify that the **atdd** command loads the bundled skill before
starting the workflow. Command discovery alone does not prove execution.
Confirm that the agent helps create missing requirements, records project
context, and waits for agreement on the selected story, criteria and scenarios.
It should demonstrate a meaningful test failure and wait again before
implementation. Also check alternative requirements locations, incomplete
backlogs, reuse of existing context and a blocked test environment.

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

## Licence

[MIT](LICENSE), copyright 2026 Utku Kopuz.
