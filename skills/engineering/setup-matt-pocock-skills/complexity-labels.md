# Complexity Labels

Two label families that answer one question: **how much model does this issue need?** `/triage` and `/to-tickets` apply them; whoever dispatches the work reads them and picks a model and an effort level before spawning an agent.

They are a separate vocabulary from the triage state roles. State says whether an issue is actionable, complexity says who should take it. Don't collapse them.

| Canonical axis | Label strings                                                       |
| -------------- | ------------------------------------------------------------------- |
| intelligence   | `intelligence: low` · `intelligence: medium` · `intelligence: high` |
| reasoning      | `reasoning: low` · `reasoning: medium` · `reasoning: high`          |

Edit the label strings to match whatever vocabulary your tracker already uses.

## The two axes are not the same thing

**Intelligence** is how much capability the change needs: how subtle the area is, how much of the system has to be held at once, how bad a plausible-looking wrong answer would be.

**Reasoning** is how much has to be worked out before the code is right. It tracks deliberation, not volume. A mechanical rename across forty files is `reasoning: low`. A five-line fix to an ordering bug is `reasoning: high`.

They come apart constantly. A tricky pure function in an isolated module is `intelligence: low` with `reasoning: high`. A cross-package rename with an obvious mechanical shape is `intelligence: high` with `reasoning: low`.

## Intelligence

**`intelligence: high`.** The change can be wrong in a way CI won't catch.

<!-- Replace these with the real ones for this repo. The best source is whatever
     your CLAUDE.md / AGENTS.md lists as rules nothing enforces, plus any area
     where a wrong answer fails silently rather than red. -->

- an area where a mistake fails quietly instead of throwing (auth scoping, row-level security, permission checks)
- schema or migration work, or a change to what a stored record means
- an identifier other systems treat as stable (a queue name, a job id, an idempotency key)
- a change spanning three or more packages, or one where the design isn't settled and an ADR may need writing

**`intelligence: medium`.** The default. One module, an existing pattern to copy, a doc that describes the subsystem, and a failure that shows up in the type checker or the test suite.

**`intelligence: low`.** Mechanical and locally verifiable. A user-facing string, a doc table row, a single-file fix whose test already exists or writes itself.

## Reasoning

**`reasoning: high`.** The correct behaviour has to be derived rather than read off the ticket. Signals: several constraints interact; anything concurrent, ordered, retried, or idempotent; a bug whose reproduction nobody has pinned down; a fix that could plausibly land in three places where the wrong choice is expensive.

**`reasoning: medium`.** The goal is clear and the implementation still has real choices to make.

**`reasoning: low`.** The ticket states the change and there is one obvious place for it.

## Judging the pair

- **Judge the axes separately.** Deciding one first and letting it drag the other is how everything ends up `high` / `high`.
- **Round intelligence up when the failure is silent.** Paying for a bigger model is cheaper than finding out in prod.
- **Round reasoning down when the brief is tight.** This is the payoff of triage: a well-specified ticket genuinely needs less deliberation than the raw issue did. Re-derive the pair after a grilling session, since grilling usually drops reasoning a level and leaves intelligence alone.
- **`high` / `high` on a fresh slice is a split signal.** In `/to-tickets`, try cutting the slice smaller before accepting the pair.
- **Default to `medium` / `medium` when genuinely unsure**, and say so in the justification. Never leave the pair off an issue that is `ready-for-agent`. An unlabelled issue is unroutable, so whoever dispatches it will guess.
- **Justify in one line** on the issue, so a human can override without re-reading the thread.

## Dispatching

Edit this table when the model lineup changes. The labels are deliberately abstract so a model swap costs one doc edit instead of a relabel of the whole backlog.

| Label                  | Model                                 |
| ---------------------- | ------------------------------------- |
| `intelligence: high`   | Claude Opus 5 (`claude-opus-5`)       |
| `intelligence: medium` | Claude Sonnet 5 (`claude-sonnet-5`)   |
| `intelligence: low`    | Claude Haiku 4.5 (`claude-haiku-4-5`) |

| Label               | Effort   |
| ------------------- | -------- |
| `reasoning: high`   | `xhigh`  |
| `reasoning: medium` | `high`   |
| `reasoning: low`    | `medium` |

The effort names don't line up one-to-one with the label names on purpose. Effort's low end is tuned for chat and classification; `xhigh` is the setting most coding and agentic work wants, so the usable range for issue work starts around `medium`. `max` stays a manual escalation for a ticket that already failed at `xhigh`, not a label value.

For an AFK agent that is a Claude Code session, both knobs are flags:

```sh
claude --model claude-opus-5 --effort xhigh
```

## Lifecycle

The pair travels with `ready-for-agent`. Moving an issue off that state makes the pair stale, so drop it or re-derive it as part of the transition. An issue in `ready-for-human` carries no complexity pair; the reason a human has it is in the brief.
