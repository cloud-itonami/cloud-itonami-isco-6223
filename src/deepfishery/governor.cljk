(ns deepfishery.governor
  "DeepSeaFisheryGovernor — the independent safety/traceability layer
  named in this repository's README/business-model.md, gating every
  vessel/crew scheduling and logistics operation an advisor may
  propose. The governor never dispatches hardware itself, never
  performs a fishing operation, and never navigates the vessel — it
  never finalizes a fishing-operation-execution decision or a voyage
  go/no-go decision in adverse conditions. This actor coordinates
  VESSEL/CREW SCHEDULING LOGISTICS ONLY. Modeled on
  cloud-itonami-isco-8350's deckcrew.governor.

  HARD invariants (:hard? true, ALWAYS :hold, never overridable):
    1. crew-member provenance     — the crew member must be
                                    independently verified/registered.
    2. no-actuation                — proposal :effect must be
                                    :propose (the governor never
                                    dispatches hardware and never
                                    performs a fishing operation or
                                    navigates the vessel itself; it
                                    only gates what the advisor may
                                    propose).
    3. vessel provenance           — the vessel must be independently
                                    verified/registered before any
                                    action.
    4. crew/vessel basis           — the crew member's registered
                                    vessel assignment must match the
                                    vessel named in the proposal.
    5. closed op-allowlist         — :op must be one of
                                    #{:log-catch-record
                                      :schedule-crew-operation
                                      :flag-safety-concern
                                      :coordinate-supply-order}. No
                                    other op is ever accepted, so a
                                    proposal can never finalize a
                                    fishing-operation-execution
                                    decision or a voyage go/no-go
                                    decision through the op itself.
    6. scope exclusion             — defense in depth against a
                                    compromised/malicious advisor:
                                    even within an allowed op, the
                                    proposal's free text must not
                                    describe directly finalizing a
                                    fishing-operation-execution
                                    decision, commencing/authorizing a
                                    voyage in adverse conditions, or
                                    overriding the vessel captain's
                                    judgment. This is phrased as
                                    finalization/execution ACTION
                                    phrases ('proceed with the fishing
                                    operation', 'override the
                                    captain's voyage decision'), never
                                    bare nouns ('fishing', 'weather')
                                    — bare nouns appear in ordinary
                                    in-scope proposals (logging a
                                    catch record from today's fishing
                                    trip, flagging a weather hazard)
                                    and a bare-noun check would
                                    self-trip on the mock advisor's
                                    own default rationale text. See
                                    `deepfishery.advisor`'s docstring
                                    and
                                    `never-self-trips-on-default-mock-advisor-proposals`
                                    below.

  ESCALATION invariants (:escalate? true, ALWAYS human sign-off
  regardless of confidence):
    7. :op :flag-safety-concern (equipment defect, weather hazard or
                                    crew fatigue concerns always reach
                                    a human).
    8. :coordinate-supply-order cost above `supply-order-cost-threshold`.
    9. low confidence (< `confidence-floor`)."
  (:require [deepfishery.store :as store]
            [kotoba.lang.text :as str]))

(def confidence-floor 0.6)
(def supply-order-cost-threshold 2000)

(def ^:private allowed-ops
  #{:log-catch-record :schedule-crew-operation :flag-safety-concern
    :coordinate-supply-order})

(def ^:private always-escalate-ops #{:flag-safety-concern})

;; Full finalization/execution ACTION phrases only — never bare nouns
;; like "fishing" or "weather" (see docstring above and
;; deepfishery.advisor's self-tripping-bug note).
(def ^:private scope-exclusion-phrases
  ["initiate the fishing operation"
   "commence the fishing operation"
   "finalize the fishing operation"
   "finalize the fishing operation decision"
   "proceed with the fishing operation"
   "commence the voyage in adverse conditions"
   "authorize the voyage in adverse conditions"
   "proceed with the voyage in adverse conditions"
   "authorize the voyage despite the forecast"
   "override the captain's voyage decision"
   "override the captain's judgment"
   "override the captain"])

(defn- scope-text [proposal]
  (str/lower (str/join " " (keep proposal [:rationale :detail :action]))))

(defn- scope-violation? [proposal]
  (let [text (scope-text proposal)]
    (boolean (some #(str/includes? text %) scope-exclusion-phrases))))

(defn- hard-violations [{:keys [proposal]} crew-record vessel-record]
  (let [{:keys [op effect vessel-id]} proposal]
    (cond-> []
      (nil? crew-record)
      (conj {:rule :no-crew-member :detail "未登録 crew-member"})

      (not= :propose effect)
      (conj {:rule :no-actuation :detail "effect は :propose のみ許可（governor は漁労・航行を直接実行しない）"})

      (nil? vessel-record)
      (conj {:rule :no-vessel :detail "未登録 vessel"})

      (and crew-record vessel-record (not= (:vessel-id crew-record) vessel-id))
      (conj {:rule :crew-wrong-vessel :detail "crew-member が別 vessel に登録されている"})

      (not (contains? allowed-ops op))
      (conj {:rule :op-not-allowed :detail "closed op-allowlist 外の op（fishing-operation-execution の直接決定や voyage go/no-go は決して許可しない）"})

      (scope-violation? proposal)
      (conj {:rule :scope-exclusion
             :detail "fishing-operation-execution decision の直接確定、adverse-conditions voyage go/no-go の直接決定、captain の判断の override は permanent hard block"}))))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `deepfishery.store/Store`. Pure — never
  mutates the store, never performs a fishing operation, never
  navigates the vessel."
  [request context proposal store]
  (let [crew-record (store/crew-member store (:crew-id request))
        vessel-record (some->> (:vessel-id proposal) (store/vessel store))
        hard (hard-violations {:request request :proposal proposal}
                              crew-record vessel-record)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        always-risky? (contains? always-escalate-ops (:op proposal))
        over-cost? (and (= :coordinate-supply-order (:op proposal))
                        (number? (:cost proposal))
                        (> (:cost proposal) supply-order-cost-threshold))]
    {:ok? (and (not hard?) (not low?) (not always-risky?) (not over-cost?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? always-risky? over-cost?))}))
