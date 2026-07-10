# Directing Multi-Instance Agent Builds with Verification Gates

*July 2026 · Alex Galle-From*

In early July 2026 I ran two multi-day software builds staffed mostly by coding-agent instances: one director-class instance writing specifications and reviewing output, and ten to fourteen worker instances implementing those specifications in isolated git worktrees. The builds produced thirteen small games and a shared persistence layer over about three days. This note documents the process controls.

The premise: an agent's report that its work is done is not evidence that the work is done. A language model summarizing its own output regresses toward the expected story — not deceit, just what summarization does. So every check has to run outside the thing being checked. Auditor independence, applied to agents.

## The setup

A human operator (me) set goals and acceptance standards. A director instance decomposed goals into written briefs — one task per worker, with scope, house style, and completion criteria. Workers executed briefs in isolated worktrees and produced, alongside code, artifacts of their own verification: screenshots, test output, replay logs. The director reviewed the artifacts before merging. I reviewed the merged result by playing it.

None of this depends on which models you use. It's a supervision structure.

## Three gates

**Gate 1 — mechanical checks, run by the director.** Build passes, tests pass — executed by the director or CI, never taken from the worker's report.

**Gate 2 — artifact review.** Every brief required visual or behavioral evidence: a screenshot of the running feature, the replay determinism log, the actual numbers. The director read these directly. This caught the failures mechanical checks can't: a feature that builds and passes tests but renders off-screen. Game entities being deleted by the physics engine after falling out of bounds — found because the director looked at a picture of the arena, not at a diff.

**Gate 3 — operator ground truth.** Every defect that survived the first two gates was found by a human using the software. "The attacks have no weight." "The button is off-screen." These reports outrank harness results, because the operator is measuring the thing you care about and the harness is measuring a proxy. Response order: fix the reported instance, understand the mechanism, then encode it as a new Gate 1 check.

## Don't sign green what's red

A gate only works if a red result is reported as red at every layer — worker to director, director to operator, operator outward. Once any layer treats a gate as a formality, the gate starts laundering unverified work as verified, which is worse than having no gate.

Standing rules from the two builds:

1. A worker that can't meet its brief reports the miss and what it tried. A truthful partial merges; a false complete doesn't.
2. Merge notes record what was verified, by whom, against what evidence. The commit log is an audit trail, not a press release.
3. No gate result gets edited to fit a deadline.

This is books-and-records discipline from regulated industries, transplanted deliberately. The evidentiary problem is the same: a third party, later, has to be able to reconstruct what was done, who authorized it, and what the checks showed.

## Two failure modes

**Workers stall waiting for supervision that isn't coming.** Four early workers finished their task and idled, waiting for some monitor process to wake them for the next step. Inside a bounded task nothing does. The fix was one sentence in every brief: no further instructions will arrive; complete the work within this run. Termination conditions belong in the specification. Most "agent got stuck" reports I've looked at since are this — an unstated protocol assumption between agent and harness, not a capability failure.

**The interesting deviations are judgment calls, not rule violations.** Late in the second build, a worker was briefed to restore a backup as part of cleanup. It noticed the operator's live save data had advanced during its run — restoring the backup per instructions would have destroyed real progress. It declined that step, said why, and finished the rest of the brief. That's the behavior you want: not compliance with a stale instruction, not silent abandonment, but a surfaced mid-run correction. An evaluation regime that scores only instruction-following punishes exactly the move that protected the user. Gates have to leave room for it — cheap for the agent to flag, visible to the reviewer. This failure class is the subject of a separate evaluation project (FCE); the point here is that it shows up unprompted in real builds.

## Cost

Director review consumed roughly one worker's worth of capacity. Briefs took longer to write than informal instructions. In exchange, across roughly two dozen worker runs, no unverified work reached the merged branch, and every regression the operator found became a durable check.

The claim is narrow. Not that the software was correct — that at every merge we knew what had been verified, how, and by whom, and when something later broke, the record showed where the gap was. Capability keeps improving on its own. Auditability is the part you have to build.

---

*Licensed [CC BY 4.0](LICENSE). Related work: the [Fiduciary Constraint Evaluation](https://github.com/substrateagnostic/fiduciary-constraint-evaluation) tests the judgment-call failure mode described above; [AgentLedger](https://github.com/substrateagnostic/agentledger) and the [Agent Evidence Record](https://github.com/substrateagnostic/agent-evidence-record) build the audit trail this note argues for.*
