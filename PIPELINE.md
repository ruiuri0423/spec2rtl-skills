# PIPELINE — moved

The coordination contract now lives as a skill, so any harness can load and invoke it like the
others: see [`skills/pipeline/SKILL.md`](skills/pipeline/SKILL.md).

It defines how the library's work runs — task-graph schedules, stepwise or concurrent execution
with one invariant loop, the capability floor and honest degradation, and the two-way ledger bus
through which the stages talk.
