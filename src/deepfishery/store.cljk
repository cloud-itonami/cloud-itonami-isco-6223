(ns deepfishery.store
  "SSoT for the ISCO-08 6223 Deep-sea Fishery Workers vessel/crew
  scheduling and logistics actor (itonami actor pattern,
  ADR-2607121000 / CLAUDE.md Actors section; README's 'Robotics
  premise' — this actor coordinates VESSEL/CREW SCHEDULING LOGISTICS
  ONLY. It never performs a fishing operation itself and never
  navigates the vessel: the fishing-operation-execution decision and
  the voyage go/no-go decision in adverse conditions stay with the
  human crew and the vessel captain's own judgment, gated behind
  this advisor/governor pair which never dispatches hardware and
  never finalizes a fishing-operation-execution decision or a voyage
  go/no-go decision). Modeled on cloud-itonami-isco-8350's
  deckcrew.store.

  Domain:

    crew-member — a registered deep-sea fishery crew member {:crew-id
                  :name :vessel-id — the vessel this crew member is
                  currently assigned/verified to}.
    vessel      — a registered deep-sea fishing vessel {:vessel-id
                  :name}.
    record      — a committed operating record (a logged catch record,
                  an accepted watch/duty schedule, a flagged safety
                  concern, or a coordinated supply order) — written
                  ONLY via commit-record!.
    ledger      — append-only audit trail, commit or hold."
  )

(defprotocol Store
  (crew-member [s crew-id])
  (vessel [s vessel-id])
  (records-of [s crew-id])
  (ledger [s])
  (register-crew-member! [s cm])
  (register-vessel! [s v])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (crew-member [_ crew-id] (get-in @a [:crew-members crew-id]))
  (vessel [_ vessel-id] (get-in @a [:vessels vessel-id]))
  (records-of [_ crew-id] (filter #(= crew-id (:crew-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-crew-member! [s cm]
    (swap! a assoc-in [:crew-members (:crew-id cm)] cm) s)
  (register-vessel! [s v]
    (swap! a assoc-in [:vessels (:vessel-id v)] v) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:crew-members {} :vessels {} :records [] :ledger []}
                                   seed)))))
