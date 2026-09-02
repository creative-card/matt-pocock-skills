---
"mattpocock-skills": minor
---

triage, to-tickets, setup-matt-pocock-skills: label issues with how much model they need, so an AFK agent can be dispatched at the right capability instead of always at the most expensive one.

Two label families, judged independently: `intelligence` (how much capability the change needs) and `reasoning` (how much has to be worked out before the code is right). They come apart constantly, so scoring them as one axis lands everything on `high`.

`/triage` proposes the pair when recommending `ready-for-agent`, re-derives it after grilling, and drops it on any transition off that state. `/to-tickets` scores at slice time, where a `high` / `high` slice is a signal to split rather than a result to publish. The agent brief repeats the pair in its header. `/setup-matt-pocock-skills` scaffolds the new `docs/agents/complexity-labels.md` config, which holds the level definitions and the table mapping each level to a model and an effort setting.
