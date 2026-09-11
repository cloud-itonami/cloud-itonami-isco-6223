(ns deepfishery.actor
  "DeepSeaFisheryActor — the ISCO-08 6223 Deep-sea Fishery Workers
  vessel/crew scheduling and logistics actor as a
  `langgraph.graph/state-graph` (ADR-2607121000 / CLAUDE.md Actors
  section). One graph run = one vessel/crew scheduling/logistics
  operation request (intake -> advise -> govern -> decide ->
  commit/hold, with a human-approval interrupt for escalated
  proposals). No infinite internal loop; checkpointed per superstep
  so an interrupted run can resume after human sign-off. Modeled on
  cloud-itonami-isco-8350's deckcrew.actor.

  ```text
  :intake -> :advise -> :govern -> :decide -+-> :commit            (:ok? true)
                                             +-> :request-approval   (:escalate? true, interrupt-before)
                                             +-> :hold               (:hard? true)
  ```

  The unconditional invariant: the Deep-sea Fishery Crew Advisor can
  never directly commit a record the DeepSeaFisheryGovernor refuses —
  every commit-record! call is gated behind `:decide`. This actor
  coordinates VESSEL/CREW SCHEDULING LOGISTICS ONLY: it never
  performs a fishing operation itself and never navigates the vessel,
  and no path through this graph can finalize a
  fishing-operation-execution decision or a voyage go/no-go decision
  in adverse conditions — the closed op-allowlist and the governor's
  scope exclusion make that structurally impossible, not merely a
  policy the advisor is asked to follow."
  (:require [langgraph.graph :as g]
            [langgraph.checkpoint :as cp]
            [deepfishery.advisor :as advisor]
            [deepfishery.governor :as governor]
            [deepfishery.store :as store]))

(defn build-graph
  "Build a compiled DeepSeaFisheryActor graph. `store` implements
  `deepfishery.store/Store`. `advisor` implements
  `deepfishery.advisor/Advisor` (defaults to `mock-advisor`).
  `checkpointer` defaults to an in-memory one."
  [{:keys [store advisor checkpointer]
    :or {advisor (advisor/mock-advisor)
         checkpointer (cp/mem-checkpointer)}}]
  (-> (g/state-graph
       {:channels
        {:request     {:default nil}
         :context     {:default nil}
         :proposal    {:default nil}
         :verdict     {:default nil}
         :disposition {:default nil}
         :record      {:default nil}
         :audit       {:reducer into :default []}}})
      (g/add-node :intake (fn [s] s))
      (g/add-node :advise
                   (fn [{:keys [request]}]
                     (let [p (advisor/-advise advisor store request)]
                       {:proposal p
                        :audit [{:node :advise :request request :proposal p}]})))
      (g/add-node :govern
                   (fn [{:keys [request context proposal]}]
                     (let [v (governor/check request context proposal store)]
                       {:verdict v
                        :audit [{:node :govern :verdict v}]})))
      (g/add-node :decide
                   (fn [{:keys [verdict]}]
                     {:disposition (cond
                                     (:hard? verdict) :hold
                                     (:escalate? verdict) :request-approval
                                     :else :commit)}))
      (g/add-node :request-approval (fn [s] s))
      (g/add-node :commit
                   (fn [{:keys [request proposal]}]
                     (let [record {:crew-id (:crew-id request)
                                    :op (:op proposal)
                                    :vessel-id (:vessel-id proposal)
                                    :payload proposal}]
                       (store/commit-record! store record)
                       (store/append-ledger! store {:disposition :commit :record record})
                       {:record record
                        :audit [{:node :commit :record record}]})))
      (g/add-node :hold
                   (fn [{:keys [verdict]}]
                     (store/append-ledger! store {:disposition :hold :verdict verdict})
                     {:audit [{:node :hold :verdict verdict}]}))
      (g/set-entry-point :intake)
      (g/add-edge :intake :advise)
      (g/add-edge :advise :govern)
      (g/add-edge :govern :decide)
      (g/add-conditional-edges
       :decide
       (fn [{:keys [disposition]}]
         (case disposition
           :commit :commit
           :request-approval :request-approval
           :hold)))
      (g/add-edge :request-approval :commit)
      (g/set-finish-point :commit)
      (g/set-finish-point :hold)
      (g/compile-graph {:checkpointer checkpointer
                         :interrupt-before #{:request-approval}})))

(defn run-request!
  "Run one operation request to completion or interrupt. `thread-id`
  scopes checkpointing for resume after human approval."
  [graph request context thread-id]
  (g/run* graph {:request request :context context} {:thread-id thread-id}))

(defn approve!
  "Human-in-the-loop resume: the interrupted `:request-approval` node
  advances straight to `:commit` on resume (approval is the act of
  resuming the thread — the human is approving the SCHEDULING/
  LOGISTICS proposal, not a fishing operation or a voyage decision,
  which this actor never performs)."
  [graph thread-id]
  (g/run* graph nil {:thread-id thread-id :resume? true}))
