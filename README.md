# ATDD Agent Plugin

A shared Acceptance Test-Driven Development (ATDD) workflow for coding agents.
Agree what success looks like, demonstrate missing behaviour with acceptance
tests, then implement and verify the result.

Use it to turn a feature request or an existing user story into testable
outcomes before changing production code. It uses your project's test tools;
there are no runtime dependencies, hooks, MCP servers or custom test frameworks.

## How it works

1. Find the requirements or help draft the selected story and acceptance criteria.
2. Propose Given / When / Then scenarios and wait for your agreement.
3. Write acceptance tests, demonstrate meaningful failures, then wait for your
   authorisation to implement.
4. Implement the agreed behaviour incrementally.
5. Run acceptance and regression checks, refactor, and record the results.

You review the intended behaviour before tests are written and the observed
failures before implementation. The plugin guides the agent through these
checkpoints; review its work as you would any other coding contribution.

## Install and start

Choose your client. Run the installation commands in your terminal, then start
a fresh agent session in the project you want to work on and send the chat
prompt shown below. Replace the example feature with your own request.

### Claude Code

```sh
claude plugin marketplace add allhandsondeck/agent-plugin-atdd
claude plugin install atdd@atdd
```

Chat prompt:

```text
/atdd:atdd-workflow help me define and build a feature for exporting records to CSV.
```

### Codex CLI

```sh
codex plugin marketplace add allhandsondeck/agent-plugin-atdd
codex plugin add atdd@atdd
```

Chat prompt:

```text
$atdd-workflow help me define and build a feature for exporting records to CSV.
```

Use `/skills` or type `$` to find **atdd-workflow** if needed.

### VS Code / Copilot

1. Run **Chat: Install Plugin from Source** from the Command Palette.
2. Enter `https://github.com/allhandsondeck/agent-plugin-atdd.git`.
3. Run **Chat: Open Customizations** and confirm **atdd** is enabled under
   **Plugins** for your chosen harness and workspace. In the Agents window,
   open **Customizations** from the sidebar.
4. In a fresh chat, select `/atdd` and describe your feature. The host may show
   a plugin-qualified command name.

Plugin support must be enabled through `chat.plugins.enabled`; availability
also depends on your harness and organisation policy. This route is packaged
but has not been verified end to end.

## Requirements and project files

You can start without a requirements document; the agent helps you draft one.
By default, it uses these files in your target project:

```text
docs/user-stories.md    Stories, acceptance criteria and scenarios
docs/atdd-context.md    Requirements source, status, test tools and setup
```

Already have requirements? Include the story and source in your request, for
example: `work on story US-003 in docs/user-stories.md`. You can supply another
file or directory. Later runs reuse the saved source unless you specify another.

The agent checks and reads the source, helps save missing requirements, and
marks unfinished setup as pending. The selected story, criteria and scenarios
must be saved and agreed before executable tests begin; the rest of your
backlog can remain incomplete. It asks only for missing information, reuses
suitable test tools and agrees new tooling with you before introducing it.

## Tested clients

| Client | Status |
| --- | --- |
| Claude Code | Installation and use checked by the maintainer on `0.1.3`. |
| Codex | Installation and use checked by the maintainer on `0.1.3`. |
| VS Code / Copilot | Packaged; complete workflow unverified. |
| Copilot CLI, Cursor and other clients | Unverified. |

Client versions were not recorded. The `0.1.4` instruction changes have not yet
been exercised in live client sessions. See the [release notes](docs/releases/v0.1.4.md)
for validation details.

## Update

For Claude Code:

```sh
claude plugin marketplace update atdd
claude plugin update atdd@atdd
```

For Codex CLI, refresh the marketplace and reinstall the package:

```sh
codex plugin marketplace upgrade atdd
codex plugin remove atdd@atdd
codex plugin add atdd@atdd
```

In VS Code, open **Plugins** and select **Check for and Apply Updates**.
Start a fresh session after updating and confirm the installed plugin version.
Changes pushed to GitHub do not directly replace an installed cached copy.

## Development and feedback

See the [development guide](docs/development.md) for package structure,
validation commands and local testing. The [shared skill](skills/atdd-workflow/SKILL.md)
contains the full workflow. Browse [GitHub releases](https://github.com/allhandsondeck/agent-plugin-atdd/releases)
for published versions.

Report installation issues or workflow surprises through
[GitHub Issues](https://github.com/allhandsondeck/agent-plugin-atdd/issues),
including your client, version and a redacted example.

## Licence

[MIT](LICENSE), copyright 2026 Utku Kopuz.
