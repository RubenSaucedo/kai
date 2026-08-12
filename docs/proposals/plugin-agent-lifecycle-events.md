# Proposal: emit subagent lifecycle events for plugin-provided agents

Status: draft, pre-implementation
Target: GitHub Copilot CLI
Author: kai maintainers
Measured against: GitHub Copilot CLI 1.0.79 (Windows)

---

## 1. Summary

The Copilot CLI emits `subagentStart` and `subagentStop` hook events for
built-in agent types and for user-defined custom agents, but **not** for custom
agents contributed by an installed plugin.

The result is that installing a plugin silently removes agent activity from
every hook consumer. The same agent definition is observable when it is copied
into `~/.copilot/agents/` and invisible when it is distributed through a
plugin. This is a behavioural inconsistency between two delivery mechanisms for
what is documented as the same kind of object.

This proposal states the problem, the evidence, the design questions the fix
has to answer, and a test plan. It deliberately does not prescribe an
implementation, because the CLI source has not been read yet.

---

## 2. Background

The hooks reference documents `subagentStart` and `subagentStop` as the
lifecycle events for delegated work, and states that built-in YAML-based agents
**and user-defined custom agents** emit them. `subagentStart` carries the agent
name; `subagentStop` carries an agent identifier and the agent's response.

Plugins can contribute custom agents. Once a plugin is installed, its agents
appear in the task tool's list of dispatchable types under a namespaced name of
the form `<plugin>:<agent>`. From the operator's point of view they are ordinary
agents: they are listed, they are selectable, they run, and they return a
response.

The documentation does not describe plugin-contributed agents as a distinct
category with different event semantics.

---

## 3. Problem statement

Hook consumers cannot observe delegated work performed by plugin-provided
agents. There is no event, no partial event, and no error, so a consumer cannot
distinguish "no agent ran" from "an agent ran and was not reported".

Three properties make this worse than a missing feature:

1. **It is silent.** Nothing in the transcript, the hook output, or any log
   indicates that an event was suppressed.
2. **It is a regression on install.** A workspace that had partial visibility
   (agents dispatched as built-in types carrying a persona prompt) loses that
   visibility entirely once the plugin is installed and its agents are used
   properly by name. Adopting the supported distribution mechanism makes the
   system less observable.
3. **It is indistinguishable from correct operation.** A consumer that reports
   "no delegated work occurred" is not detectably wrong.

---

## 4. Evidence

All three cases below were run in a single sandbox `COPILOT_HOME`, with one
hook installed, in the same workspace, within minutes of each other. The only
variable is how the agent reaches the CLI.

| Agent | Delivery | `subagentStart` / `subagentStop` |
| --- | --- | --- |
| `explore` | built-in type | emitted |
| `rubber-duck` | built-in type | emitted |
| `probe-security` | file in `~/.copilot/agents/` | emitted, name `probe-security` |
| `kai:principal-swe-architect` | installed plugin | **none** |
| `kai:principal-security` | installed plugin | **none** |

The user-defined case and the plugin case used **the same agent definition
file**; only the name and location differed. The plugin agents ran correctly
and returned responses, so this is an eventing gap, not a dispatch failure.

### 4.1 Reproduction

```bash
# 1. Isolate a home so the result cannot be contaminated by existing config.
export COPILOT_HOME="$(mktemp -d)"

# 2. Install any plugin that contributes custom agents.
copilot plugin install <owner>/<repo>

# 3. Register a hook on subagentStart and subagentStop that appends its
#    payload to a file.

# 4. Dispatch a built-in agent  -> events appear.
# 5. Copy one plugin agent file into "$COPILOT_HOME/agents/", rename it,
#    dispatch it                -> events appear, with the real agent name.
# 6. Dispatch the same agent by its plugin name "<plugin>:<agent>"
#                               -> no events at all.
```

Steps 4-6 must be run in one session to rule out hook-loading differences.

---

## 5. Why this matters beyond one plugin

Anything built on these events inherits the gap:

- **Audit and compliance.** A record of which agents acted on a repository is
  incomplete, and incomplete in a way that cannot be detected from the record.
- **Observability and telemetry.** Duration, frequency, and failure-rate
  metrics undercount by an unknown amount that varies with plugin adoption.
- **Cost and usage attribution.** Delegated work attributed to no agent.
- **Guardrails and policy.** A hook that exists to notice, gate, or annotate
  delegated work does not fire for exactly the agents an organisation chose to
  install deliberately.
- **Plugin ecosystem incentives.** A plugin author currently faces a real
  trade-off between distributing agents through the supported mechanism and
  keeping them observable. That trade-off should not exist.

The common factor is that hooks are the CLI's supported extension point for
observing agent activity, and a supported distribution mechanism bypasses it.

---

## 6. Proposed direction

**Plugin-provided custom agents should emit `subagentStart` and `subagentStop`
on the same terms as user-defined custom agents.** Event emission should be a
property of dispatching an agent, not a property of where its definition was
loaded from.

Design questions the implementation must answer, to be settled against the
source:

1. **Where does emission actually live?** Determine whether emission is keyed
   to a resolved agent record shared by all sources, or is attached to specific
   dispatch paths. This decides whether the fix is a one-line source-agnostic
   change or a genuine refactor. *This is the first thing to read.*

2. **What name goes in the payload?** The namespaced `<plugin>:<agent>` is
   unambiguous across plugins and matches what the operator typed. Emitting the
   bare name would collide with same-named user agents. Recommendation: emit the
   namespaced name as the agent name, and consider adding an explicit
   `pluginName` field rather than asking consumers to parse a colon.

3. **Is any existing consumer relying on the silence?** Consumers will start
   receiving events for agents they never saw. This is the intended outcome and
   the fix is not useful without it, but it should be called out in the release
   notes as a behaviour change rather than a pure bug fix.

4. **Do other events have the same gap?** The audit should not stop at these
   two. Any event carrying an agent identity should be checked against all
   three delivery mechanisms, since a fix that repairs one path and leaves
   another is the same bug with a smaller blast radius.

5. **What is the failure mode if a plugin agent name is malformed?** A hook
   payload should never carry an unvalidated plugin-supplied string into a
   consumer without a documented shape.

### 6.1 Non-goals

- Changing the hook payload schema for existing emitters.
- Adding new event types.
- Changing how plugin agents are resolved, namespaced, or dispatched.
- Any behaviour specific to one plugin.

---

## 7. Test plan

The regression that allowed this should be closed by a test that is
parameterised over delivery mechanism, not by a test for plugin agents
specifically:

- For each of {built-in type, user-defined agent, plugin-provided agent},
  dispatch an agent and assert that `subagentStart` and `subagentStop` are both
  emitted, and that the documented payload fields are present and non-empty.
- Assert the emitted agent name for the plugin case is the namespaced form.
- Assert that a plugin agent whose definition is invalid does not emit a
  half-formed event.

A table-driven test makes the next delivery mechanism fail loudly instead of
silently.

---

## 8. Open items before writing code

- [ ] Read the dispatch path and locate every emission site.
- [ ] Confirm on Linux and macOS that this is not Windows-specific.
- [ ] Confirm against the latest `main`, not only 1.0.79.
- [ ] Check whether the documentation intends plugin agents to be covered, or
      whether the documentation should change instead. If the current behaviour
      is deliberate, the reason belongs in the docs, because the operator-facing
      surface gives no hint of the distinction.
- [ ] Decide payload naming (item 2 in section 6) before opening the PR, since
      it is the only part that is hard to change afterwards.

---

## 9. Status of claims in this document

Measured on 1.0.79: the evidence table, the reproduction, and the
observation that plugin agents dispatch and return normally.

Not yet verified: anything about the CLI's internal structure, whether the
behaviour is intentional, and whether it reproduces on other platforms or on
`main`. Section 6 is a direction, not a design.
