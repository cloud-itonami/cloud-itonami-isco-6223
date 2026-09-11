# cloud-itonami-isco-6223

Open Occupation Blueprint for **ISCO-08 6223**: Deep-sea Fishery Workers.

This repository designs a forkable OSS business for deep-sea fishery
vessel/crew scheduling and logistics coordination: a crew-scheduling and
equipment-logistics robot manages watch/duty schedules, catch-record
logging and supply coordination under a governor-gated actor, so a fishing
vessel operator keeps its own crew scheduling and audit records instead of
renting a closed maritime crewing SaaS.

**Maturity: `:implemented`.** `src/deepfishery/` implements the
`DeepSeaFisheryActor` as a `langgraph.graph/state-graph`
(`deepfishery.actor`) wired to a `Deep-sea Fishery Crew Advisor`
(`deepfishery.advisor`) and an independent `DeepSeaFisheryGovernor`
(`deepfishery.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok?) +-> :request-approval (:escalate?, human-in-the-loop interrupt)
+-> :hold (:hard?)`. See `kbb -M:test` output for the current
test/assertion count.

HARD invariants (always hold, never overridable): crew-member and vessel
provenance (both must be independently verified/registered before any
action), no-actuation (`:effect` must be `:propose`), the crew member's
registered vessel assignment must match the vessel named in the proposal, a
closed op-allowlist (`:log-catch-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — no other op is ever
accepted), and a scope-exclusion check that permanently blocks any proposal
whose free text describes directly finalizing a fishing-operation-execution
decision, a voyage go/no-go decision in adverse conditions, or overriding
the vessel captain's judgment. Always-escalate (human sign-off regardless
of confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)): `:flag-safety-concern`
(always) and `:coordinate-supply-order` above the registered cost
threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot
performs the physical domain work**. Here a crew-scheduling and
equipment-logistics robot performs watch/duty scheduling, catch-record
logging and supply-order coordination under an actor that proposes actions
and an independent **Deep-sea Fishery Governor** that gates them. The
governor never dispatches hardware itself and never performs a fishing
operation or navigates the vessel; `:high`/`:safety-critical` actions (a
flagged safety concern, or a supply order above the registered cost
threshold) require human sign-off.

**This actor coordinates VESSEL/CREW SCHEDULING LOGISTICS ONLY.** It never
performs a fishing operation itself and never navigates the vessel.
Deep-sea fishing crews work in open ocean, exposed to fall-overboard risk,
extreme-weather conditions and isolation from rescue — the
fishing-operation-execution decision, and any go/no-go decision to sail or
continue a voyage in adverse conditions, stay with the human crew and the
vessel captain's own judgment. The closed op-allowlist and the governor's
scope-exclusion check make it structurally impossible for this actor to
finalize a fishing-operation-execution decision, decide a voyage go/no-go
in adverse conditions, or override the captain's judgment — those are
always a hard, permanent block, never an op this actor can propose or
auto-commit.

## Core Contract

```text
crew roster + vessel assignment + watch/duty request + catch/supply request
        |
        v
Deep-sea Fishery Crew Advisor -> DeepSeaFisheryGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a fishing-operation-execution decision, decide a voyage go/no-go
in adverse conditions, override the vessel captain's judgment, or suppress
an operating record without governor approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `6223`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
