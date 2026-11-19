---
name: implement-spec
description: "Implement the result of /to-spec and /to-tickets in code."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker and the complexity label config should have been provided to you. If the issue tracker wasn't, tell the user to run `/setup-matt-pocock-skills`.

The goal is the entire spec implemented on a single **integration branch**, with every ticket resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

Start every subagent with a fresh context window.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

The run is **autonomous**: the user expects to come back to a finished integration branch, not a queue of questions. Decide everything you can, through the proxy where a decision is the user's to make, and loop the user in only on a **critical decision** (see below).

## Steps

1. Read the spec and tickets to understand the task graph.

2. Grill every `ready-for-human` ticket through the proxy (see **Proxy grilling**) so it can join the frontier. Start with the ones that block the most.

3. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

4. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 6 (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

5. Use a fresh **implementer subagent** for each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls the Skill tool with `tdd` to build the ticket;
   - merges the integration branch tip into its own branch before reporting done

   Dispatch it at the model and effort its ticket's complexity pair maps to (see **Dispatching within the budget**).

6. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**.

7. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

8. Once all tickets are complete, call the Skill tool with `code-review` on the integration branch. Fix all issues raised by the code review in a single fresh **implementer subagent**.

9. If a draft PR exists, mark it ready for review. Otherwise, resolve each ticket the way the issue tracker closes work. Either way, report the integration branch, every answer the proxy marked as a guess, and any ticket still waiting on the user.

10. Clean up all **implementer subagent** worktrees.

## Dispatching within the budget

Every subagent spends the user's subscription. Spend it where the work needs it:

- **Implementers** run at the model and effort their ticket's complexity pair maps to in the complexity label config's dispatch table. A ticket without a pair (one created mid-run, or one the proxy just grilled) gets a pair derived against that config before dispatch, `medium` / `medium` when unsure. With no config at all, judge the same two axes yourself: how much capability the change needs, and how much has to be worked out.
- **Other subagents** take the smallest model that does the job: the merger at `intelligence: low` unless the merge conflicts, exploration at `medium` / `medium`, the review-fix implementer at the highest pair among the tickets its findings touch, and the proxy at `intelligence: high`, since its wrong answers fail silently.
- **A ticket that fails at its level** is retried once, one level up on the axis that failed it.
- **Concurrency** is capped by the budget as well as the frontier. When the harness reports a usage limit, stop dispatching, let running subagents finish, and resume after the reset rather than downgrading models to squeeze more in.

## Proxy grilling

A `ready-for-human` ticket is human for one of two reasons. It needs a judgement call or design decision, which the proxy can make, or it needs something only a human can do (credentials, an external dashboard, manual testing), which it can't.

1. Start a **proxy subagent** whose job is to answer as the user would. Point it at everything that records how the user decides: the user's memory files, any wiki the user or a `chief-of-staff` session keeps, the repo's ADRs, glossary and agent instructions, the spec, and a short note of the user's stances you've seen in this session. It cites the source behind each answer, marks an answer with no source as a **guess**, and answers **escalate** for a critical decision.
2. Grill the ticket against the proxy: call the Skill tool with `grilling` and `domain-modeling`, and put each question to the proxy instead of the user. Resume the same proxy across questions where the harness allows.
3. Write the result as an agent brief on the ticket (`../triage/AGENT-BRIEF.md` has the format), listing the proxy's guesses, then derive its complexity pair and move it to `ready-for-agent`.

A ticket of the second kind stays `ready-for-human`. Name it in the final report with what the user has to do.

## Looping the user in

A **critical decision** is one whose wrong answer is expensive to undo or reaches past this spec:

- hard to reverse: a migration, deleting data, a change to what a stored record means
- outward-facing: a public API or contract other systems rely on, anything published, sent, or paid for
- security, auth, or permissions
- it contradicts an ADR, the spec, or a decision the user has recorded
- it changes the plan: a ticket's scope grows or shrinks materially, or a ticket turns out to be unnecessary

Everything else, decide yourself and record the decision and its reason on the ticket, so the user can audit it after the run. For a critical decision, hold only the tickets that depend on it, keep the rest of the frontier moving, and bring the user the question, the options, and your recommendation. Batch questions that arrive close together into one message.
