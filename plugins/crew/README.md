# crew

A team of subagents for working with Claude Code under *supervised autonomy*
(level 2): the agents do the work, you approve the plan and validate the result.

Works on server/web codebases and on native iOS and Android projects. The agents
run on the filesystem, the terminal and git — not inside an IDE — so the same
pipeline works from VS Code, from a plain terminal, or in CI.

> The agent prompts are written in Spanish. Everything else — manifests,
> commands, this documentation — is language-neutral. The pipeline itself works
> regardless of the language you write your issues in.

## When NOT to use the pipeline

The full team is 8 subagents, and each one runs its own requests. For a typo, a
rename or a one-line fix it costs far more than it contributes: the flow triages
first and handles trivial changes directly. The pipeline is for features and for
multi-file or security-sensitive changes.

## Measured results

Real issues from two production codebases — a TypeScript monorepo (React +
Fastify + Postgres) and a native iOS app (Swift/SwiftUI + SwiftData + Firebase)
— measured with Claude Code's own usage panel. These are percentages of total
consumption, not absolute tokens.

| | #36 auth | #21 content | #45 privacy | #22 screens | iOS #8 notifications |
|---|---|---|---|---|---|
| Plugin version | initial | 0.5.0 | 0.6.0 | 0.6.1 | 0.11.0 |
| Agent dispatches | 9 | 21 | 11 | 20 | 19 |
| `coder` passes | 2 | 5 | 1 | 5 | 1 |
| Correction rounds | 1 (manual) | 3 | 0 | 2 | 2 |
| Blocking findings | 3 | 9 | 0 | 9 | 9 |
| `coder` (% of usage) | 40% | 10% | 10% | 29% | 31% |
| **Orchestrator** | 5% | **33%** | 26% | **4%** | 1% |
| Context >150k | 50% | 59% | 58% | **36%** | 35% |
| `architect` | didn't run | didn't run | 1% | 3% | ~1% |
| Plugin (% of usage) | 63% | 74% | 67% | 76% | 86% |

**Implementation is expensive; supervision is cheap.** One `coder` call cost
roughly 7x a security audit and 20x a plan. Hence the design: security
constraints are pinned in the PLAN, not in the audit afterwards. In #45
`architect` cost **1% of usage** and caught four discrepancies in the plan —
including one that would have shipped broken — before a single line was written.
`coder` ran once instead of five times.

**That the review earns its keep, measured.** In #21 it found two critical
authorization failures (any `viewer` could publish and delete through the API
directly) plus seven highs, all fixed by the loops before reaching the human.

**The loop's second pass pays for itself.** Across seven consecutive runs, the
re-audit that follows the fixes found something the first review had missed: two
high findings with a proof of concept in #45, an unauthenticated DoS that took
down the whole API process in #22, and — most telling — in iOS #8 it found that
the *previous round's fix did not actually work on the real code paths*. A
continued auditor would have confirmed its own finding closed. Capping the loop
at a single pass would have let all of them through.

**The orchestrator can overflow too.** In #21 coordination became the single
largest cost (33%) and the session was cut off mid-repair. Handing findings to
`coder` in batches by area, instead of all at once, brought it down to 4% and
cut heavy context from 58% to 36%.

**A negative result, so this doesn't oversell.** The plugin pins security
constraints in the plan and gives `coder` an explicit checklist (no defaulted
secrets, random tokens, nothing sensitive in the clear). In #22 those exact three
classes of bug showed up anyway, and the audit afterwards caught them — not the
checklist. Detection is what demonstrably works; that early prevention reduces
rework remains unproven.

**A rule that backfired, and how it showed.** Version 0.9.0 allowed continuing an
existing agent for bounded confirmations, justified as "it preserves context".
That reasoning sounds good everywhere, and the orchestrator generalized it: one
`coder` carried a full implementation plus seven repair batches, chained by
message. Context >150k hit 80% and the orchestrator climbed back to 24%. A
continued agent's context only grows and is reprocessed every turn. Fixed in
0.12.1 by writing down the limit, not just the reason.

**Honest caveats.** The tasks are not equivalent: #21 was full-stack with new
surface area, #45 was frontend on an already-audited base, so part of the drop to
zero findings is the size of the change, not the plugin. These are a handful of
runs, not a controlled experiment. And `security-auditor` charges its ~10% every
time, whether it finds anything or not: it is an insurance premium, not a cost
that pays back on every run.

**The pipeline is not your biggest expense.** Across every measurement, the
dominant consumption was 8+ hour sessions and contexts above 150k, not the
subagents. Use `/clear` when switching tasks before you start trimming agents.

## Pipeline

```
product-analyst -> plan -> [GATE 1: you approve the plan]
                     |                  |
          architect hardens it          | requires changing the plan?
          (adds the security            |      -> stop, back to GATE 1
           constraints a generic        |
           planner omits)               v
                              coder -> tester <--+
                                          |      | fails: debugger (max 2)
                                          +------+
                                          |
                                      tests green
                                          |
                                          v
                         reviewer + security-auditor + docs-writer
                             (in parallel, over the same diff)
                                          |
                                  blocking finding? --yes--> fix + re-test
                                          |                    (max 2 passes)
                                          no
                                          v
                              [GATE 2: you validate the diff]
```

The flow corrects itself before handing you anything: it won't give you a diff
with red tests or with unaddressed blocking findings. Whatever it can't resolve
in 2 passes is escalated to you — and if that leftover is itself in a blocking
category, it arrives as an explicit decision, not as a bullet in a list of
pending items. Minor findings are reported without being fixed. If a fix requires
changing the approved plan, the loop stops and returns to gate 1: the redesign is
yours to approve.

**Blocking** means critical/important from `reviewer`, critical/high from
`security-auditor`, and — regardless of the label it arrives with — anything that
crosses a tenant or account boundary, or disables a security escape hatch
(revoke, disable, degrade privacy). Those are graded by the boundary they break,
not by how hard they are to reach: "there's no UI that triggers it today" is not
a mitigation.

## Platform support

The team detects the project type during triage and applies the matching rules.

| | Verification | Static analysis |
|---|---|---|
| Server / web | bring the stack up (`docker compose`, dev server) and exercise it end-to-end | `tsc`, ESLint |
| iOS | `xcodebuild test` against a simulator — headless, Xcode never has to be open | `xcodebuild build`, SwiftLint |
| Android | `./gradlew test`, `connectedAndroidTest` on an emulator | `assembleDebug`, ktlint/detekt |

On mobile, `coder` applies an additional security checklist on top of the
server-side one: the binary can be decompiled so no secret ships inside it, the
app is never a trust boundary, sensitive data goes to Keychain or encrypted
storage rather than the default store, and the surface exposed to the OS (URL
schemes, universal links, exported components) is validated.

Anything requiring privileged platform interaction — signing, provisioning,
certificates, physical-device testing — is escalated to you by design, never
automated.

## Subagents

| Agent | When it runs | Touches code |
|---|---|---|
| product-analyst | First: clarifies requirements and acceptance criteria | No |
| architect | Produces the technical plan, or hardens an existing one | No |
| coder | Implements the approved plan | Yes |
| tester | Writes and runs tests, verifies against the real system | Yes |
| reviewer | Reviews the diff for quality before delivery | No |
| debugger | Root cause + minimal fix when something breaks | Yes |
| docs-writer | Updates documentation after a change | Docs only |
| security-auditor | Security pass over the diff on sensitive changes | No |

## How it got here

Every version below came from a failure observed in a real run, not from
speculation. The one exception is marked.

| Version | What it fixed |
|---|---|
| 0.4.0 | Bounded correction loops |
| 0.5.0 | Triage, security moved into the plan, verification against the real system |
| 0.6.0 | Adaptive `architect`, aligned severity scales, findings handed over in batches |
| 0.7.0 | Severity graded by the boundary broken, not by exploitability |
| 0.8.0 | Compile + static analysis added to the exit criteria |
| 0.8.1 | `Closes #N` when opening a PR |
| 0.9.0 | Cheap re-verification vs. independent audit |
| 0.9.1 | The closing keyword is never translated — GitHub only accepts English |
| 0.10.0 | Approval required before creating issues, same as for a PR |
| 0.11.0 | Platform-agnostic: native iOS and Android *(not derived from a measurement)* |
| 0.12.0 | Hitting the loop cap doesn't downgrade a blocking finding |
| 0.12.1 | Agent continuation restricted to small confirmations |

## Components

- `agents/` — the 8 subagents.
- `skills/level2-workflow/` — the orchestration discipline (plan -> validation).
- `commands/kickoff.md` — the `/crew:kickoff <description>` command that fires
  the full pipeline.
