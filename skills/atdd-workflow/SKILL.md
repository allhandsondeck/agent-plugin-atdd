---
name: atdd-workflow
description: Develop software from agreed user stories and acceptance criteria using a test-first ATDD workflow. Use when the developer requests Acceptance Test-Driven Development (ATDD) or an acceptance-test-first workflow for a feature or behaviour change.
---

# ATDD workflow

Establish what success looks like and how it will be verified before changing
production code. Keep the workflow proportional to the requested behaviour.

## Terminology

- **User story:** who needs something, what they need and why.
- **Acceptance criterion:** an observable condition the story must satisfy.
- **Acceptance scenario:** a concrete Given / When / Then example illustrating
  one or more acceptance criteria.
- **Acceptance test:** executable verification of those outcomes.

## Project context

On first use, inspect existing project guidance, dependencies, test
configuration and relevant implementation before asking setup questions.
Ask only about missing information and unresolved choices. Create a small
`docs/atdd-context.md` in the target project to record:

- The concrete requirements file or directory, its verified status and how
  stories are identified. If the source has not yet been created or checked,
  mark it as pending; a default-path instruction is not a verified source.
- The relevant stack and application boundaries being tested.
- The agreed acceptance test tools, test locations and commands.
- Required environment setup, services and test data, linking existing
  documentation rather than duplicating it. Do not store secrets.

Reuse this context on later runs. Check it against the current project and
refresh changed information with the developer; do not treat saved choices
as permanent or repeat a questionnaire whose answers are already available.

Prefer the project's existing tools when they can verify the agreed outcomes.
Choose the test boundary from the behaviour: a browser for a user journey, an
API for an HTTP contract, or a message or domain interface where appropriate.
Do not select a tool solely because a project is labelled frontend or backend.
If the project has no suitable tooling, explain the smallest viable option
and agree it with the developer before introducing a framework. Record the
agreed verification approach in the context file.

## 1. Understand requirements

Resolve the requirements source in this order: the developer's supplied file
or directory, the concrete source saved in `docs/atdd-context.md`, then the
default `docs/user-stories.md` at the target project's root. On every run,
check that the chosen path exists and read its relevant contents. For a
directory, locate and read the file containing the selected story. A saved
path or a generic instruction to use the default is not evidence that the
requirements exist. If an existing source cannot be read, resolve that
problem rather than treating it as missing or replacing it.

Record the actual location and observed status in the context file. Work
directly with an existing source without copying it into a competing
requirements document. Preserve its structure and identifiers where possible.

If requirements are missing, help create them interactively. Start with the
requested behaviour, ask focused questions and draft a user story with
acceptance criteria. Do not invent product rules or speculate about the
entire product. Save the draft story and acceptance criteria in the chosen
requirements source, creating its parent directories when needed. Read back
the saved content and update the context with its actual path and status;
do not leave a missing source represented as completed setup. A requirements
document containing a sufficiently defined selected story is a prerequisite
for the test-first implementation cycle; other stories can remain incomplete.

By default, collect all known product stories in `docs/user-stories.md` rather
than creating a separate document for each feature. Give each story a stable
identifier, a description of who needs what and why, and separately identified
acceptance criteria. Existing identifiers need not be renamed. Make criteria
precise and testable, covering applicable rules, permissions, boundaries,
alternative outcomes and failures. Detail should resolve ambiguity, not add
unrelated requirements or prescribe internal implementation.

Select the story for this cycle and clarify its scope with the developer.
Do not require the whole product backlog to be specified before starting.

## 2. Define acceptance scenarios

Add scenarios beneath the selected story's acceptance criteria in the chosen
requirements source. Follow existing scenario organisation when using an
alternative source. Keep a clear connection between story, criterion,
scenario and executable test; one test can cover multiple criteria if it
verifies their outcomes explicitly.

Write concrete Given / When / Then scenarios in domain language. Include the
main outcome and relevant alternatives or boundaries. Describe observable
behaviour rather than private methods or implementation structure. For example:

```text
Scenario: Reject a withdrawal exceeding the available balance
Given an account has an available balance of £20
When the account holder requests a withdrawal of £30
Then the withdrawal is declined
And the available balance remains £20
```

For each scenario, propose how it will be tested using the agreed tools and
an appropriate public boundary, such as a UI, API or domain interface.
Markdown scenarios do not require a Gherkin runner or a new test framework.

Present the selected story, criteria, scenarios and any unresolved questions.
**Wait for explicit agreement on these and the verification approach before
writing executable tests or implementation.** A request to start ATDD does not
itself agree to newly drafted requirements or scenarios. If the developer
changes agreed behaviour later, update the source and obtain agreement on
the changes before continuing affected work. Existing explicit agreement in
the current conversation can be reused; do not ask for it again unchanged.

## 3. Verify acceptance tests fail

Before writing executable tests, read the saved requirements source and verify
that it contains the selected story, agreed acceptance criteria and scenarios.
If any are missing or incomplete, return to the requirements and scenario
steps. A context entry or a draft only in chat does not meet this prerequisite.

Translate the agreed scenarios into executable acceptance tests using the
project's conventions. Keep assertions connected to the agreed outcomes; do
not change production behaviour yet. Check relevant existing tests so that
pre-existing failures can be distinguished from the new failures.

Run the new tests and inspect why they fail. Establish that the failure exposes
the missing behaviour. Broken imports, syntax errors, missing dependencies and
unavailable services do not establish a behavioural failure. Resolve test setup
problems before claiming the red stage is complete.

If the environment prevents execution, record the blocker and stop to resolve
it with the developer. Do not substitute imagined results or proceed with
implementation. If a scenario already passes, report that evidence; do not
break working code to manufacture a failure. Clarify any remaining work with
the developer before changing production behaviour.

Record test paths, commands and observed failures under the selected story in
the requirements document, or link evidence in the project's existing format.
**Pause after demonstrating the intended failing tests and wait for the
developer to authorise implementation.**

## 4. Implement incrementally

After authorisation, implement one agreed behaviour at a time. Write only the
code needed to satisfy it, rerun the relevant tests and proceed to the next
scenario. Add supporting lower-level tests when useful. Avoid unrelated
refactoring and speculative abstractions.

Preserve the agreed outcomes. Do not weaken assertions, skip tests or change
expected results simply to make the implementation pass. Resolve a genuine
requirement change with the developer through the scenario agreement step.

## 5. Verify and refactor

Run all agreed acceptance tests and the project's relevant regression checks.
Refactor where it improves the changed code, then rerun affected checks. Keep
the agreed behaviour passing throughout refactoring.

Update the selected story's verification evidence with the final commands and
observed results, linking criteria and scenarios to their tests. Distinguish
passed, failed and blocked or unverified outcomes. Report what changed and any
remaining limitations. Do not claim completion while an agreed criterion or
scenario for this cycle remains failing or unverified.
