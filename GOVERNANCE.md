# Governance

`cloud-itonami-isco-6223` is an OSS open-occupation blueprint. Governance
covers both code and the operator model.

## Maintainers

Maintainers may merge changes that preserve these invariants:

- the Advisor cannot directly dispatch robot actions, perform a fishing
  operation or navigate the vessel.
- DeepSeaFisheryGovernor remains independent of the advisor.
- hard policy violations cannot be overridden by human approval.
- the closed op-allowlist never grows to include an op that could finalize
  a fishing-operation-execution decision, decide a voyage go/no-go in
  adverse conditions, or override the vessel captain's judgment.
- every commit, hold and approval path is auditable.
- real crew/vessel operator data stays outside Git.

## Decision Records

Architecture decisions live in `docs/adr/`. Changes to the trust model,
storage contract, public business model, operator certification or license
should add or update an ADR.

## Operator Governance

Anyone may fork and operate independently. itonami.cloud certification is a
separate trust mark and should require security, audit, support and
data-flow review.

Certified operators can lose certification for:

- bypassing policy checks
- widening the op-allowlist to include a fishing-operation-execution
  finalization, voyage go/no-go, or captain-override op
- mishandling crew/vessel operator data
- misrepresenting certification status
- failing to respond to security incidents
- hiding material changes to customer-facing operation
