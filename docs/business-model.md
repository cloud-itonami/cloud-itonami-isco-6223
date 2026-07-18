# Business Model: Deep-sea Fishery Crew Scheduling and Logistics Coordination

## Classification

- Repository: `cloud-itonami-isco-6223`
- ISCO-08: `6223`
- Occupation: Deep-sea Fishery Workers
- Social impact: maritime-safety, crew-welfare, catch-data-transparency

## Customer

- deep-sea fishing vessel operators
- fishing/crewing agencies
- independent deep-sea fishery crews

## Offer

- watch/duty scheduling coordination
- catch-record (catch-quantity/species/trip-log) data logging
- safety-concern escalation (equipment defect, weather hazard, crew fatigue)
- vessel/equipment supply-order coordination (ice, bait, safety gear, net
  repair supplies)

## Revenue

- monthly retainer
- per-vessel scheduling fee

## Trust Controls

- crew-member and vessel provenance verified before any action
- no ledger record without an independently registered crew member and
  vessel
- safety concerns always escalate to human sign-off regardless of confidence
- supply orders above the registered cost threshold always escalate to
  human sign-off
- this actor coordinates VESSEL/CREW SCHEDULING LOGISTICS ONLY — it never
  finalizes a fishing-operation-execution decision, decides a voyage
  go/no-go in adverse conditions, or overrides the vessel captain's
  judgment; those are a permanent, non-overridable hard block enforced by
  a closed op-allowlist and an independent scope-exclusion check
- scheduling and logistics records are auditable, not editable
