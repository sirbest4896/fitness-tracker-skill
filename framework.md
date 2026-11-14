# Framework — BestFit (F1–F5, Data Model, Goals Model)

This is the data layer underneath an unchanged five-section design (Today / Train / Calendar / Macros / More). Nothing in this file changes a screen; it changes what the records underneath the screens can guarantee.

Design basis, in plain terms:

- **Sync** follows offline-first practice: stable never-reused IDs, a revision counter per record, an idempotency key per mutation, tombstones for deletes, field-level ownership, and fuzzy matches routed to an ambiguity queue — never auto-merged.
- **Completeness** follows the product owner's rule (Decision, 14 Nov 2026 — see `DECISIONS.md`): three states per day; a macro day is `complete` only when breakfast, lunch and dinner are all logged, or the user confirms the day — the food-source refresh never writes completeness, and the system never closes or locks a day on its own, so an incomplete day stays open; training-day completeness is **user-asserted only**; missing meals are asked about gently and suspected-partial days are flagged and asked about; averages run over complete days only, with the denominator printed. The canonical statement is §F3.
- **Provenance** follows nutrition-assessment practice: every value carries its source, its `provStatus` (observed / estimated / derived), and its revision; corrections keep the original visible in history.
- **Progression** is computed from set-level records; the e1RM convention recorded for future surfacing is the mean of the Epley and Brzycki estimates, from work sets of 1–10 reps only, with warm-ups, bodyweight, assisted and changed-machine sets kept as separate, non-comparable series. (Surfacing progression is out of scope for this version; the data must support it.)

---

## F1 — Identity & sync contract

Applies to sessions and macro logs (and, by extension, any synced record):

- Add to each record: `uid` (stable, never reused), `revision` (integer, increments on every change), `updatedAt`, `deletedAt` (tombstone; null while active).
- Unique constraint on `(source, externalRef)`.
- **Match order on sync:** externalRef → uid → fuzzy key.
  - A fuzzy match with a changed date (a reschedule) updates the one record and logs the move in the sync log and record history.
  - A fuzzy match that cannot be resolved safely goes to the **ambiguity queue** and is put to the user. It is never auto-merged and never silently duplicated.
- **Idempotency:** every mutation carries an `idempotencyKey`. Replaying the same mutation (same payload, same key) is a no-op that returns the existing record. Submitting the same macro payload twice with the same key returns the same record id both times — exactly one record.
- **Field ownership:**
  - Source-owned fields (sync may update): date, time, title, status = scheduled / cancelled.
  - User-owned fields (sync may **never** overwrite): logged exercises, performed sets, weights, user corrections, completed status.
  - Consequence: a re-sync carrying an empty exercise list can never blank a session the user has already logged.
- **Deletes are tombstones.** A cancellation sets `deletedAt` / cancelled state; the record remains distinguishable from a never-existed record, and a stale source re-sending the pre-cancellation payload cannot resurrect it. A supported restore path returns the record to active with its full history intact.
- **Restore trigger (normative — Gate G3 clarification P2).** An incoming synced payload for a tombstoned `(source, externalRef)` **restores** the record when it is a genuine new booking event: the payload has `cancelled: false` **and** its date/time differs from the tombstoned record's last state **and** (where the source supplies an event/update timestamp) that timestamp is later than the cancellation event. An identical payload, or one whose timestamp is earlier than or equal to the cancellation event, is stale: the outcome is `tombstoned-kept` and nothing changes. A restore sets `deletedAt` back to null, keeps the **same `uid`**, increments `revision`, and appends a `record_history` row recording the transition `cancelled → restored`; the sync outcome name is `restored`. (This is the only reading consistent with the `(source, externalRef)` uniqueness rule above: a re-booking under the same externalRef can never become a second record.) A user-confirmed restore from a backup snapshot is a separate surface — `restoreFromSnapshot` in the Actions section — and follows the same history discipline.
- **Duplicate guards at write time:** a session write rejects a logged weight of 0 (0 is never a logged weight), and rejects an exact duplicate (same date + title + exercises).
- **Scheduled-only gate:** any record created from a calendar or email source is created with status scheduled. Only the user's attendance confirmation can set it completed; completed status is user-owned from then on.

Defects this contract closes (rationale, for the builder): reschedule duplication under a date-based de-dupe key; sync overwriting logged exercises with an empty payload; macro saves stacking duplicate days; hard deletes allowing resurrection; last-writer-wins conflicts with no revision to compare; silent sync failures with no log.

## F2 — Provenance

- **Canonical field name: `provStatus`.** (Decision O6, `DECISIONS.md`. The alternative name seen in older framework text is not used anywhere in this product.)
- `provStatus` values, exactly three:
  - `observed` — a value the user gave, or a value synced from a source the user consented to, as-is.
  - `estimated` — a value produced as an estimate (for example from a meal description), only where the user granted estimates permission, always labelled an estimate to the user.
  - `derived` — a value computed by the product from other recorded values (for example a calculated macro target, or the weight trend line).
- User-facing label mapping (onboarding vocabulary → data value):
  - "user-given" → `observed`
  - "calculated" → `derived`
  - "estimate" → `estimated`
- Every macro log, and every session field group, carries: `source`, `provStatus`, `sourceRef`, `revision`.
  - `source` values are enumerated — no other values may be written: `chat`, `calendar`, `email`, `manual`, `food_tracker`, `health_connect`, `apple_health`, `pre_upgrade_import`.
  - **Food-source keys** (the `sourceKey` on the `macro_refresh` consent row, bound in onboarding Block 8f): exactly one of `food_tracker` (a paired food tracker / food log), `manual` (the user reads out or dictates day totals; the "refresh" is then a chat write with source `manual`), or `none` (no food source paired; the macro refresh does not run). A paired food tracker's day writes carry `source = food_tracker`.
  - **Steps-source keys** (the `sourceKey` on the `steps_sync` consent row, picked in onboarding Block 8e): exactly one of `health_connect`, `apple_health`, `manual`, or `none`. Steps writes carry that key as their `source` (for `manual`, source `manual`).
  - `sourceRef` identifies the originating record or day (for a food-source day total, the date; for a booking, its booking reference where one exists).
- **`record_history` table:** on every correction, the prior value is kept: which record, which field(s), prior value, new value, source of the correction, and when. A correction replaces the live value with the user's corrected value and scales nothing else silently; the original remains visible in history. Provenance is a field for this purpose: a `provStatus` transition is itself written as a `record_history` row (Gate G3 clarification, observation O-j), so the audit trail covers provenance changes, not only value changes.
- **Item-level estimates:** where a food day is built from items, an estimate flag at item level is preserved into the day's provenance: a day whose entries are estimates is written with `provStatus = estimated`; a day of user-given figures is `observed`.
- **Import honesty:** records imported from an older format gain provenance `observed` with source recorded as a pre-upgrade import — labelled honestly, never back-invented with a more specific source than the import actually knows.

## F3 — Day completeness

- Fields per macro day and per training day: `completeness` ∈ {`none`, `partial`, `complete`}, plus `completenessProvStatus` ∈ {`observed`} (observed = the state rests on logged meals or on the user's own assertion; no system process assigns completeness, so completeness is never a derived value), `suspectedPartial` (boolean) and `suspectedPartialReason` (text, present when the flag is set). A day that is not `complete` is **open**: it is never locked, never expires, and can be completed at any later time by logging the missing meals or by the user's confirmation.
- **Canonical completeness rule — this paragraph is normative and every other file in this package states the same rule in substance (the product owner's decision, 14 Nov 2026, `DECISIONS.md`).** A food day is `complete` **only** when breakfast, lunch and dinner are all logged, **or** the user confirms the day (for example confirming a meal was skipped, or that the day is done). When all three meals are logged, the day is complete **by that fact** — the meals being logged is what completes it; no job declares it complete. The macro refresh may compute and show that all three meals are present, but it **never writes completeness**: a day's completeness changes only from logged meals or from the user's confirmation. The system **never closes or locks a day on its own** — a day with missing meals stays **open** (state `none` or `partial`, never locked), stays editable indefinitely, and completes whenever the missing meals are later logged or the user confirms it. Missing meals are asked about **gently, in the user's own tone** — "What did you have for lunch?", never the rigid "Did you have lunch?" — ask and wait; if still unanswered, ask again the next day. Training-day completeness is **user-asserted only** — the system never sets a training day complete on its own. Any user assertion about a day is kept in record history, and the system never silently changes a user-asserted state. Separately, the system flags suspected-partial days (with a reason) and asks about them; flagging changes no state by itself. In any weekly summary or reconciliation, a day with unlogged meals is flagged **not complete**, naming what is missing (for example "you didn't log lunch on Tuesday"), and is never silently treated as complete.
- **System role:** flag suspected-partial days and ask — gently ("What did you have for lunch?"), ask and wait, and ask again the next day if still unanswered. The system never writes a day's completeness itself.
- **Suspected-partial rule (normative — Gate G3 clarification P1).** The two signals are: logging stops early in the day, and the day's total is far below the user's own median for complete days. Operationally:
  - The **complete-day median** is the median of daily calorie totals over **complete days in the last 28 days**. It requires **at least 3 complete days** in that window; with fewer, the calorie signal is unavailable and is never guessed.
  - A partial day is flagged `suspectedPartial` when **(a)** at least one meal is logged and at least one meal is missing, **and (b)** the day's calorie total is **below 50% of that median**, **or** the only logged meal is breakfast. The calorie signal alone, with all meals logged, never flags — a low-total complete day is complete by fact.
  - **Evaluation points:** the flag is evaluated on every `saveMacroLog` write affecting that date, and recomputed at `getAskQueue` composition for the requested date. **The writer is the macro-log write path / queue read**, which stores `suspectedPartial` and `suspectedPartialReason` on the macro-log record. The macro refresh job does **not** write the flag (consistent with its never-writes-completeness rule).
  - `suspectedPartialReason` names the signal(s) in plain words (for example "only breakfast logged; total is about a sixth of your usual complete day").
  - Flagging never writes `completeness` and never closes a day.
- **Training-day completeness criteria (normative):** a training day is `complete` when the user has confirmed an attendance outcome (completed / skipped / rescheduled) for **every** session scheduled that day, or the user logs a completed session for that day with nothing scheduled still awaiting an outcome; it is `partial` when **some but not all** of that day's scheduled sessions have a confirmed outcome and no completed session settles the day; it is `none` when nothing is logged and no outcome is confirmed. A session left in scheduled status at day end therefore keeps the day out of `complete` — it is never auto-completed.
- **Averages rule:** all averages, hit rates and multi-day chart denominators are computed over **complete days only**, and always printed with coverage, in the pattern: "over N complete days; P partial days excluded; M days not logged".
- Partial days remain visible wherever the day itself is shown, labelled partial. Blank days are skipped — never zero-filled, never averaged in.
- Rationale stated once, plainly: a partially logged day averaged as if full distorts every figure built on it for far longer than the day itself; a blank day costs nothing but honesty.

## F4 — Tracking data model additions

### performed_sets

Written alongside the existing session record (which stays as the display source — no screen changes; progression becomes computable):

| Field | Notes |
|---|---|
| `id` | row id |
| `sessionUid` | the session's stable `uid` |
| `exerciseName` | as logged |
| `exerciseVariant` | the confirmed machine/setup variant where known; blank if unconfirmed |
| `setNo` | set number within the exercise |
| `reps` | as given (string preserved where the user gave a range) |
| `weight` | as given; blank if unknown; never 0 as a logged weight |
| `unit` | as given, labelled |
| `isWarmup` | warm-up flag; warm-up sets are a separate, non-comparable series |
| Framework fields | `uid`, `revision`, `updatedAt`, `deletedAt`, `source`, `provStatus`, `sourceRef` |

### body_metrics

| Field | Notes |
|---|---|
| `date` | user's local day |
| `weight` | nullable; user weigh-ins only (or a consented source the user chose) |
| `bodyFat` | nullable |
| `steps` | nullable; written by the steps sync / manual step logs; never 0-filled for a missing day |
| `water` | nullable |
| `sleep` | nullable; user-supplied values only in this version (no automatic sleep sync) |
| Framework fields | `uid`, `revision`, `updatedAt`, `deletedAt`, `source`, `provStatus`, `sourceRef` |

- **Population (Decision O4):** `body_metrics` starts **empty** in every generic install. It is populated by: the steps sync (steps), manual weigh-ins (weight and any other metric the user supplies), and manual step logs. There is **no seeding from any spreadsheet** in the generic product.
- **Write-preservation rule:** a steps write for a date preserves any weight (and other metrics) already recorded for that date; a weight write preserves existing steps. Each write touches its own metric fields only.
- The older intent to seed this table from pre-existing imported history is **not** a generic-product behaviour and is not claimed here: in the generic product the table simply starts empty and fills from use.

### Mapping fix (part of F4)

The exercise → muscle mapping runs in the exact order in `SKILL.md` / `catalog.md`. In particular, Rear Delt Fly / Reverse Fly / Reverse Pec Deck are matched **before** the generic fly/pec rule, and Wrist Curl and Leg Curl are matched **before** generic "curl". Regression cases that must always pass: Wrist Curl → Forearms only; Reverse Pec Deck → Rear Deltoids only (and a plain Pec Deck still → Pectorals); Leg Curl → Hamstrings only.

### Heat-map data (part of F4)

`muscle_states` carries, per muscle: the manual override (value + timestamp) where one exists, plus `hasHistory` (boolean) and `lastWorked` (date or null) derived from completed sessions. Never-logged is therefore "no data yet" in the data (`hasHistory = false`, `lastWorked = null`), distinguishable from recovered. Display behaviour is specified in `design-specs.md` and is deliberately unchanged by this data distinction. **Known behaviour (Gate G3 clarification, observation O-d):** muscle states are maintained at write time — updated when a session is logged or an override is set — not re-derived on read; reads return the maintained state as of the session timeline, which is the honest semantics for a cooldown model.

### Weight trend (framework position)

The stored data is the raw weigh-ins in `body_metrics`. The displayed trend is a derived value under the EWMA-lite rule decided in `DECISIONS.md` (alpha 0.3 over logged weigh-ins, full-history computation, line breaks at gaps). The trend is never written back over raw values.

## F5 — Confirm-before-save transaction

Every save — onboarding, session logging, macro logging, corrections, settings and consent changes — follows this transaction:

1. **Parse** the user's input into a proposed record.
2. **Propose** the record to the user (read-back of what will be saved).
3. **One material clarification** if needed (for example the Photo Rule for an ambiguous machine). The clarification is bound to the draft it belongs to; nothing else in the draft is blocked by it once resolved.
4. **Confirmation bound to that exact revision:** the user confirms the proposed record as shown. The onboarding/save form of the confirmation question is exactly: "Ready for me to save this? Reply yes to save, or tell me what to change." Save only on an unambiguous yes to that question; a "yes" answering any other question is never consent.
5. **Atomic save** of exactly the confirmed revision.
6. **Read-back receipt** in chat: what was saved.

Rules:

- **Any edit voids the earlier confirmation.** After an edit, the changed parts are read back and confirmation is asked again.
- **Retries cannot duplicate** — the save carries an idempotency key (F1).
- **Enforcement split (state this honestly):** the retry/idempotency half is enforced by the app actions. The edit-invalidates-confirmation half is enforced by the chat flow as a process rule; it is not an app-level transaction property, and the product must not claim it is.
- **Unconfirmed items stay unlogged (the product owner's decision, 14 Nov 2026 — `DECISIONS.md` O5):** nothing is logged unless the user confirms it. If the agent is unsure about any part of a draft (which machine is the main case), it asks in chat first, in confirmation form ("Is this the one?"). If the user does not reply, the item stays **unlogged and unlocked** as a pending confirmation, and that pending confirmation is carried into the nightly Ask Queue in the **last position**, asking again. Still unconfirmed after that: still not logged — keep waiting. A pending confirmation never expires into a guess.

---

## Full data model

Framework fields on every synced/correctable table unless noted: `uid`, `revision`, `updatedAt`, `deletedAt`, `source`, `provStatus`, `sourceRef`. Dates are the user's local days (YYYY-MM-DD) in their onboarding timezone. Sets/reps/weight on exercise items are strings, so blanks, units and ranges like "8-12" survive intact.

### Evidenced tables (from the source product)

| Table | Fields |
|---|---|
| `baselines` | group, exercise, weightLabel, note. (Fresh template seed: weight 0 = "not set yet", treated as blank by all logic.) |
| `workout_plans` | name, focus, dayLabel, notes, exercises[] |
| `sessions` | date, title, type, status (scheduled / completed / skipped), timeLabel, notes, exercises[] (each item `{name, sets, reps, weight, note}`), externalRef, suspectedPartial (+reason). Training-day completeness is **not** a per-session column: its home is a per-day aggregate over the day's sessions, computed under the §F3 criteria and exposed by the Train read contract in the Actions section (Gate G3 clarification P3) |
| `macro_target` | calories, proteinG, carbsG, fatG, waterMl, notes — all nullable; plus provenance for how the target came to exist (user-given → observed; calculated → derived) |
| `macro_logs` | date + the same value fields as `macro_target`, all nullable; plus completeness, completenessProvStatus, suspectedPartial (+reason) |
| `muscle_states` | manual overrides only, plus (F4) `hasHistory`, `lastWorked` per muscle |
| `notes` | See schema note below |
| `performed_sets` | F4 table above |
| `body_metrics` | F4 table above |
| `record_history` | See schema note below |
| Sync log | entries: timestamp, source, record reference, action taken (created / updated / moved / tombstoned / tombstoned-kept / restored / refused), failure flag + reason where failed. Read via the sync-log read action; also the basis for sync-health (last-success per source, stale warning after 48 hours without success where a source is expected) |

Schema notes (closing an evidence gap — these column sets are **PRD-designed** for this generic product, stated so a builder can build them; they are not claimed as extracted from any existing install):

- `notes`: `id`, `date` (nullable), `text`, framework fields.
- `record_history`: `id`, `recordUid`, `recordTable`, `field`, `priorValue`, `newValue`, `changedAt`, `source`, `provStatus`.
- `ambiguity_queue`: `id`, `createdAt`, `source`, `incomingSummary`, `candidateRecordUid` (nullable), `status` (open / resolved-linked / resolved-new / resolved-discarded), `resolvedAt`. An entry is created by F1 when a fuzzy match cannot be resolved safely; resolving it is a user decision under F5.

### Actions

**Evidenced read actions:** `getOverview`; `getTodayDashboard`; `getMorningBrief`; `getMacroSummary` (returns per-metric averages plus a `coverageLabel`); `getStepsSummary`; `getWeightSummary`; `getRecommendations`; `getAskQueue` (takes a date, returns the queue items for that day); `getSyncLog`; `getFrameworkStatus` (active/tombstoned counts and history rows; also exposes each automation job's `owner` and `lastSuccessAt` from the `automation_consents` record — Gate G3 clarification P4).

**Read-action contracts for the two composite reads (PRD-designed shapes, normative for builders):**
- `getMorningBrief(date)` — the read the morning-brief job composes from. Inputs: one local date. Returns: `{ date, missingMeals: [meal names still unlogged for the previous day], chaseNeeded: boolean, summaryPoints: [exactly 3 strings, each derived from logged data], numbersLine: { yesterday: {calories, proteinG, carbsG, fatG, steps, weight} (each nullable), sevenDay: {per-metric averages over complete days only, coverageLabel} }, recommendation: { text, basis } | null, goalMode, sourcesUnavailable: [names] }`. It writes nothing; a null recommendation means the honest on-track state is used by the caller.
- `getAskQueue(date)` — returns `{ date, items: [item] }` where each item is `{ slot: 0|1|2|3|4, kind: 'missing_food'|'attendance'|'suspected_partial'|'blank_weights'|'pain_readiness'|'pending_confirmation', prompt: string, recordRef: uid|null, missingMeals: [names] (slot 0 only), sessionUid: uid|null (slot 1), suspectedPartialReason: string|null (slot 2), exerciseNames: [names] (slot 3), muscle: string|null (slot 4), pendingDraftUid: uid|null, confirmationKind: string|null (for example 'machine_pick') and photoPairKey: string|null (pending_confirmation only; photoPairKey set only for a machine pick) }`. At most 5 items, in slot order, with the pending-confirmation rule of `DECISIONS.md` O5 applied by this read: a `pending_confirmation` item, when present, takes the last position (`slot: 4`) for that night, displacing the lowest-priority present slot if all five are filled (slot 0 is never displaced). An empty `items` array means the Ask Queue job sends nothing. **Oldest-first pending confirmations (known behaviour, stated — Gate G3 clarification, observation O-b):** when more than one pending confirmation exists, this read surfaces only the **oldest** one (one `pending_confirmation` item per night); the younger item surfaces on a following night once the older one resolves. Nothing is logged either way; the rule is specified here so it is deliberate, not accidental.
- `getTodayDashboard(date)` — **as-of semantics (Gate G3 clarification, observation O-c):** the dashboard answers **as of the requested date**. Its summary points (last session, next up, and any similar point) are computed from records dated on or before the requested date; a fact dated after the requested date is never cited for that date.
- **Macro-log read surface (Gate G3 clarification, observation O-a):** every macro-log read (the per-day reads behind `getMacroSummary`, the Macros section and the dashboard) surfaces, per day, `provStatus`, `completenessProvStatus`, `source`, `sourceRef` and `revision` alongside the values and the completeness fields. A normative F2/F3 field that no read can show is unverifiable and is not acceptable.
- **Train read — per-day training completeness (Gate G3 clarification P3):** the Train section's data read returns, alongside its existing payload, `trainingCompleteness`: one entry per day that has scheduled or logged sessions in the requested range, each `{ date, state (none / partial / complete), basis }`, where `basis` names the §F3 criterion in plain words (for example "1 of 2 scheduled sessions has a confirmed outcome"). The state is the per-day aggregate defined in §F3 — user-asserted only; no system process sets it.
- `getReconciliationReport(weekStart)` — **PRD-designed read (Gate G3 clarification P5).** Read-only; changes nothing. Returns the Job 6 reconciliation report structure (`automation.md`) for the week starting `weekStart`, so the report can be produced and inspected in an install without a live cron: `{ weekStart, countsMatch: boolean, duplicates: [day/record references], missingDays: [dates], mismatchedTotals: [{ date, sourceTotal, trackerTotal }], failedSyncs: [{ source, reason, at }], notCompleteDays: [{ date, missingMeals: [meal names] }] }`. The weekly job's delivered report and this read present the same comparisons; neither fixes anything.

**Evidenced write actions:** `saveMacroLog` (upsert by date when called with no id; idempotent under its idempotency key); `saveStepsLog` (upsert of `{date, steps, source, provStatus, sourceRef}`; preserves same-date weight); `saveSession` (rejects 0 as a logged weight; rejects an exact duplicate of date + title + exercises); `upsertSyncedSession` (creates/updates a synced session as scheduled-only, under the F1 contract — it may not write user-owned fields; for a tombstoned `(source, externalRef)` it applies the §F1 restore trigger: a genuine re-booking returns the outcome `restored` — same `uid`, `deletedAt` cleared, `revision` incremented, a `cancelled → restored` history row appended — and a stale payload returns `tombstoned-kept` with no change); `seedDuplicate` (build/restore helper; re-invoking it on an already-seeded install returns a not-seeded result with zero rows added).

**PRD-designed actions** (required to build the product end-to-end; they close the evidence gap recorded at Gate G0 as G4, and are design decisions for this generic product, not claims about an existing install): `saveBaseline`; `savePlan`; `saveBodyMetric` (weight and other user-supplied metrics; preserves same-date steps); `setMuscleOverride`; `applyRecommendations` (the Apply + Confirm write for the Recommendations block); `saveGoals` (the goals record below); `saveAutomationConsent`; `saveProfile`; `resolveAmbiguity` (records the user's queue decision and applies it under F1/F5); `createBackupSnapshot` (below); `restoreFromSnapshot(snapshotId)` (Gate G3 clarification P2: restores the records of the named backup snapshot into a scratch/restore state by **idempotent replay/read-back** — replaying it twice changes nothing the second time, and it never destructively wipes a live install; as a write it is user-confirmed under F5, and restored records keep their `uid`s and history. This is the surface AC-18 / D8 is verified through. A manual un-tombstone of an individual record by the user, if offered, goes through the same history discipline as the §F1 restore). Every PRD-designed write action follows F1 (idempotency key, revision) and F5 (confirmation binding) exactly like the evidenced ones.

**Backup / manifest creation procedure (PRD-designed, normative — closes Gate G1 gap B4):** `createBackupSnapshot` is run by the system (a) **weekly, immediately before the reconciliation job runs**, (b) **after any framework change**, before the change-recheck procedure, and (c) on the user's request. It creates one snapshot containing: `createdAt`; per-table active and tombstoned counts (the manifest counts reconciliation and the record-preservation diff compare against); the list of macro-log dates with each day's `revision` and `completeness`; the macro target record's `revision`; the session list (uid, date, status, revision) — **session entries must reflect each record's true state, including tombstone**: a tombstoned session is listed as tombstoned, never under its pre-cancellation status such as `scheduled` (Gate G3 clarification, observation O-g); and a checksum over that exported overview. The snapshot is stored **inside the install's own Space data** as a `backup_snapshots` row (manifest fields above plus the exported overview payload), never outside the install, and snapshots are append-only — creating a new one never overwrites an older one. Reconciliation and restore both read the **latest snapshot taken before the period under review** unless a specific snapshot is named.

---

## Goals data model (PRD-designed)

The records below are **designed in this PRD** to hold what `onboarding.md` collects. They are marked PRD-designed because no source table for them exists in the audited material; a builder implements them exactly as specified here.

### user_profile

| Field | Notes |
|---|---|
| `displayName` | nullable; as given in onboarding Block 1 |
| `timezone` | as given; all dates are local days in this timezone |
| `unit` | `kg` or `lb`, as given; never silently converted |
| `smithMachine` | yes / no / unsure, as given (onboarding Block 5) |
| `smithBarWeight` | nullable; value + unit as given, or null = "blank — not known" |
| `machinesUsedOften` | free text, nullable |
| `trainingTypes`, `sessionsPerWeek`, `split` | as given, nullable |
| `trainingDaysFit` | the days / pattern that fit the user's week, as described by them, nullable (onboarding Block 2); a life pattern, never converted into a schedule they did not agree to |
| `mealStructure` | the meals that make up the user's day, as recorded at onboarding (the packaged prompt or Block 7); default breakfast / lunch / dinner. Drives the gentle missing-meal asks (the user is only ever asked about meals in their structure) and the day view. Completeness itself is unchanged: §F3 / the product owner's Decision 1 — complete only when breakfast, lunch and dinner are all logged, or the user confirms the day, never written by a job; `mealStructure` governs the asks and the day view only, never the completeness test |
| `connectorChoices` | free-text summary of the Stage B connector record: food source (name + key), steps platform (key), calendar (connected / not), email (connected / not); nullable |
| `injuriesAsStated` | verbatim text, or "none stated" |
| `setsRepsScheme` | as given or accepted-as-suggestion, nullable |
| `foodReportingStyle` | exact numbers / app totals read out / meal descriptions, nullable |
| `estimatesPermission` | boolean; true only where the user granted it in onboarding Block 7 |

### goals

One current goals record per install (history of changes kept in `record_history`):

| Field | Notes |
|---|---|
| `goalOwnWords` | verbatim; never shortened or replaced by the mode label |
| `trackerWants` | verbatim; what the user wants from the tracker itself, in their own words (onboarding Stage C); never shortened or rewritten. The agent's capability mapping of those wants is a separate note in `record_history`, never an edit to this field |
| `goalMode` | `lose_fat` / `recomposition` / `gain_muscle` / `gain_weight` / null (blank) |
| `targetWeight` + `targetWeightUnit` | nullable; never suggested or computed for the user |
| `pacePreference` | verbatim text, nullable |
| `stepGoal` | nullable integer; blank if not given |
| `success12Weeks` | verbatim text, nullable |
| Framework fields | `revision`, `updatedAt`, `source` (= onboarding or later chat change), `provStatus` (= observed for user-given values) |

### Goal-mode maths (normative for this product)

- Maintenance estimate: `maintenanceKcal = latestWeightKg × stepFactor`, where `stepFactor` comes from the user's average daily steps under the factor table decided in `DECISIONS.md` (O1). **Step-factor averaging window (normative): the average is the mean of the user's logged step days in the last 14 days; days with no step data are excluded (never counted as 0); a minimum of 3 logged step days in that window is required — with fewer, the step factor, and therefore maintenance, is unavailable.** There is no separate non-step "activity picture" input in this product: the step average as just defined is the only activity input. If the user's weights are stored in lb, convert the single weight value to kg for this calculation only, show the conversion in the working, and store nothing converted. If there is no current weight or the step average is unavailable, maintenance is unavailable.
- Mode targets: Lose Fat `maintenanceKcal × 0.85`; Recomposition `× 0.97`; Gain Muscle `× 1.07`; Gain Weight `× 1.12`.
- Split basis weight (Decision O3): `basisWeightKg = targetWeight if set, else latest current weight, else calculation unavailable`.
- Split presets (protein g/kg, fat g/kg, on the basis weight): Balanced 2.1 / 0.95; High Protein 2.8 / 0.85; High Carb 1.9 / 0.7; Low Carb 2.2 / 1.4. Sliders adjust the protein and fat g/kg values from the preset starting point.
- Kcal factors (Decision O2, standard Atwater, PRD-decided): protein 4 kcal/g, carbs 4 kcal/g, fat 9 kcal/g.
- Carbs are always the remainder: `carbsKcal = targetKcal − proteinG×4 − fatG×9`; `carbsG = max(0, carbsKcal / 4)`. Carbs never go negative; a clamp at 0 is shown in the working.
- **Rounding order (Gate G3 clarification, observation O-h):** protein and fat grams are rounded to whole grams **first**; carbs are then computed from those rounded values. (All hand-checked cases in testing were internally consistent under round-grams-first; the order is stated so future builds match.)

### goal_recommendation_state

The display-only state behind the Recommendations block and the onboarding calculation:

| Field | Notes |
|---|---|
| `maintenanceKcal` | derived; null when unavailable |
| `goalMode` | copied from `goals` at computation time |
| `preset` | balanced / high_protein / high_carb / low_carb |
| `proteinPerKg`, `fatPerKg` | slider values (g/kg) |
| `basisWeightKg` + `basisKind` | `target` or `current` — which basis O3 used |
| `recommendedCalories`, `recommendedProteinG`, `recommendedCarbsG`, `recommendedFatG` | derived results; `provStatus = derived` |
| `unavailableReason` | null, or which required input is missing |
| `computedAt` | timestamp |
| `appliedAt` | null until the user does Apply + Confirm; applying copies the recommended values into `macro_target` as a confirmed write under F5 and records the change in `record_history`. Nothing else writes `macro_target` from this state. |

### automation_consents

One row per job (job keys: `ask_queue`, `morning_brief`, `booking_calendar`, `booking_email`, `macro_refresh`, `steps_sync`, `reconciliation`):

| Field | Notes |
|---|---|
| `jobKey` | as above; calendar and email booking consent are **separate rows, separate answers** — one answer is never copied onto the other |
| `owner` | the agent or user role responsible for the job (Gate G3 clarification P4); recorded when consent is set up, nullable until then |
| `lastSuccessAt` | the job's last successful run, per job and — for source-bound jobs — per source (Gate G3 clarification P4); null until the job first succeeds; updated by the job as it runs. This is the record the sync-health behaviour and `getFrameworkStatus` read |
| `enabled` | boolean; default false for every job at install |
| `localTime` | the user's chosen send/run time in their timezone; nullable |
| `doNotSend` | true when `enabled` is true but no time was chosen — the job is recorded on-but-not-sending until a time is picked (onboarding Block 8 rule) |
| `deliveryChannel` | a placeholder binding set at install (the user's chosen chat); never a shipped value |
| `sourceKey` | for source-bound jobs, the single source the user picked, from the enumerated key lists in §F2 (food: `food_tracker` / `manual` / `none`; steps: `health_connect` / `apple_health` / `manual` / `none`), or none |
| `intervalMinutes` | booking rows only: the mirror watch interval in minutes (a duration, not a clock time). Template default offered at onboarding: **30 minutes**, recorded only as the value the user confirms or changes in onboarding Blocks 8c/8d; null until confirmed |
| Framework fields | `revision`, `updatedAt` |

Sync health and change recheck have no consent rows: sync health is a behaviour of the consented jobs plus the sync log; change recheck is an event-triggered procedure after a framework change (see `automation.md`).

---

## Invariants (must hold for every install, at all times)

1. No record exists without provenance (`source`, `provStatus`, `sourceRef`, `revision`) where this framework requires it.
2. No average is produced without its coverage denominator.
3. No sync write touches a user-owned field.
4. No delete is a hard delete on a synced record.
5. No target changes except by: user-given value, accepted calculated value at onboarding, or Apply + Confirm.
6. No automation exists without a consent row; no consent row defaults to enabled.
7. A 0 baseline is read as "not set" by every piece of logic, without exception.
