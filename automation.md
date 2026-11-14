# Automation — BestFit (Job Templates)

Every job in this file is a **template**. It ships with placeholders, not with any user's configuration:

- `{{user_timezone}}` — the timezone recorded in onboarding (asked at Stage A / Block 8, before the job times, and confirmed in Block 1).
- `{{delivery_chat}}` — the chat the user chose for tracker messages, bound at install. This package ships no chat identifier of any kind.
- `{{ask_queue_time}}`, `{{morning_brief_time}}`, `{{macro_refresh_time}}`, `{{steps_sync_time}}`, `{{reconciliation_time}}` — times **the user sets in onboarding Block 8** (or later, by telling the agent). There are no shipped default times. Descriptive defaults in words only: the Ask Queue runs in the **evening**, the macro refresh and steps sync run **after** the Ask Queue in the evening, the reconciliation runs **weekly in the evening**, and the morning brief runs in the **morning, just after the user's own daily brief if they have one**. Onboarding resolves those words into the user's actual chosen times.
- `{{food_source}}` — the user's food record, bound in onboarding Block 8f as a source key from the enumerated list in `framework.md` §F2: `food_tracker` (a paired food tracker / food log), `manual` (totals the user reads out), or `none` (no pairing; Job 3 does not run).
- `{{booking_mirror_interval}}` — the booking mirror's watch interval in minutes (a duration, not a clock time). The template offers a generic default of **30 minutes**; the value is recorded only as the one the user confirms or changes in onboarding Blocks 8c/8d, and is stored as `intervalMinutes` on the booking consent rows.
- `{{steps_source}}` — the single steps source the user picked in onboarding Block 8e (a health platform, manual, or none).
- `{{tracker_space}}` — the install's own tracker Space.

**Consent gate (applies to every job):** a job exists only where its `automation_consents` row is enabled (see `framework.md`). A job recorded on with no time chosen is **do-not-send** until the user picks a time. Turning a job off, changing its time or changing its source is done by telling the agent, read back and confirmed under F5.

**Onboarding position (new-user experience, product owner's decision 14 Nov 2026):** these jobs are introduced **first** in onboarding (Stage A — Block 8 runs before connectors, basics and goals), because the user should hear what the automation does before anything personal is asked. Each job is explained on its own (what it does, when it runs, what it sends), each is opt-in separately at the user's own time, and every job here is a template the user can re-time, re-source or switch off at any time — nothing about a job's schedule is locked at onboarding.

**Instance times:** one existing install's clock times are that install's configuration. They are not defaults, they are not in this file, and an implementer must not reintroduce them as defaults. A worked configuration, if an implementer needs one for testing, uses mock times chosen for the test.

---

## Job 1 — Booking mirror (booking sync)

- **Consent keys:** `booking_calendar` and `booking_email` — two separate consents, asked separately in onboarding (Blocks 8c and 8d). Either, both, or neither may be on.
- **Cadence:** a repeating watch over the consented sources every `{{booking_mirror_interval}}` minutes — template default **30 minutes**, confirmed or changed by the user at onboarding (Blocks 8c/8d) and stored as `intervalMinutes`. The behaviour plus this parameter is the spec.
- **Path (PRD-specified, Decision O7):**
  1. A booking appears in a consented source (calendar entry or email confirmation/change/cancellation).
  2. The mirror reads it and extracts: date, start time where present, title/venue, booking reference where present.
  3. It writes to the **tracker calendar** as a session via the F1 contract (`upsertSyncedSession`): status **scheduled only**, source = `calendar` or `email`, `externalRef` = booking reference where one exists, idempotency key from the source record.
  4. A change from the source updates the one record's source-owned fields (a move is a move of the same record, logged). A cancellation tombstones it.
  5. Only the user's later attendance confirmation can make the session completed (F1 field ownership).
  - Note, stated honestly: this end-to-end path is **specified by this PRD** so the product is buildable as one coherent flow, and it is consistent with every evidenced fragment (scheduled-only arrivals, the F1 contract, the mirror pattern). It is recorded as a specification decision in `DECISIONS.md`, not as a claim about how any particular existing install is wired internally.
- **Unclear bookings:** if details are unclear or conflicting, the job does **not** write; it messages the user via `{{delivery_chat}}` with what was found and what is missing, and holds for confirmation.
- **Pings:** only on a conflict, an unclear booking, or a cancellation of a previously mirrored booking. Silent when nothing new arrives.
- **Never:** books, changes or confirms anything with an outside provider. It mirrors; it does not act on the source.

## Job 2 — Ask Queue (evening)

- **Consent key:** `ask_queue`. **Time:** `{{ask_queue_time}}` (evening, user-chosen). **Delivery:** one message to `{{delivery_chat}}`.
- **Inputs:** the tracker Space's ask-queue read for today's date (`{{user_timezone}}`); a cross-check of `{{food_source}}` / chat logging for meals the tracker does not yet have, so the missing-food question names only meals still missing (the later macro refresh will write confirmed food into the tracker).
- **Read-only:** this job writes to no tracker. It asks; the user's answers are logged afterwards through the normal flows.
- **Personal context:** where the user's records hold their stated goals and what they want from the tracker (the goals record and the verbatim wants), a question may reference them **in the user's own words** where it genuinely connects — as personal context, never as generic motivational template text, and never invented when the records hold none.
- **Output:** ONE message, headed as a quick check-in, questions numbered, one line each, **maximum 5 questions**, in this slot order:
  - **Slot 0 — Missing food (always first; never dropped when food is unlogged).** Ask **gently, by meal name** — "What did you have for lunch?", never the rigid "Did you have lunch?". If no food is logged today: ask what the user had for breakfast, lunch and dinner. If some meals are logged: ask only for the missing meal(s), by name. Ask and wait; a meal still unanswered is asked again the next day, and the day stays open meanwhile. The user's answers are later logged to `{{food_source}}` with the usual verified count, which feeds the tracker macro row via Job 3.
  - **Slot 1 — Attendance.** For each of today's sessions still in scheduled status: completed, skipped, or rescheduled?
  - **Slot 2 — Suspected-partial food day.** For a day flagged suspected-partial: complete it, mark it partial, or estimate the missing meal (only with the user's approval, and the estimate is labelled estimated).
  - **Slot 3 — Blank-weights roundup.** This week's exercises logged without a weight: fill in any the user remembers; the rest stay blank. Asked once, no pressure.
  - **Slot 4 — Pain / readiness.** A muscle the user marked sore more than 3 days ago: still sore, or clear it? Restrictions never silently expire into "ready".
- **Pending confirmations (Decision O5, as decided by the product owner, 14 Nov 2026):** nothing is logged unless the user confirms it. An unconfirmed item — a machine pick under the Photo Rule is the main case, but the rule covers anything the agent is unsure about — is asked in chat first, in confirmation form ("Is this the one?"), at the moment of logging; that in-chat flow is its home. If the user does not reply, the item stays **unlogged and unlocked**, and the pending confirmation (queue item kind `pending_confirmation`, see `getAskQueue` in `framework.md`) is carried into this queue for that night in **position last (slot 4's place)**, asking again. When a pending confirmation exists it takes the last position; if all 5 slots are filled, it **replaces the lowest-priority present slot** for that day — slots are displaced in reverse priority order (slot 4 first, then slot 3, and so on), and slot 0 (missing food) is never displaced. Still unconfirmed after that night: still not logged — keep waiting, and the pending confirmation may be asked again on a following night. It never expires into a guess. With no pending confirmation, no such item appears in the queue.
- **Ping rule:** if the queue is completely empty (food fully logged, no scheduled sessions awaiting attendance, nothing flagged), the job finishes with **no message**. The missing-food question is un-droppable **conditional on food actually being unlogged** — it is not sent when the day is fully logged.

## Job 3 — Macro refresh (evening, after the Ask Queue)

- **Consent key:** `macro_refresh` — a separate consent row with its own user-chosen time, collected in onboarding Block 8f together with the food-source binding. The job runs only where that row is enabled (and the bound food source is not `none`). **Time:** `{{macro_refresh_time}}` (evening, after the Ask Queue).
- **Trigger / inputs:** today's totals from `{{food_source}}` (its overview / day totals for today's date in `{{user_timezone}}`).
- **Write:** upsert the day into the tracker via the macro-log save (no id → upsert by date), with:
  - calories / protein / carbs / fat from the food-source totals;
  - `source` = the food source key (recorded as `food_tracker`);
  - `provStatus` = `observed`, or `estimated` if the day's food entries are estimates;
  - `sourceRef` = the date;
  - the day's meal presence is computed and may be shown (which of breakfast, lunch and dinner are present in the source day, missing meals recorded as the reason a day is not complete) — but this job **never writes `completeness`**. Under the canonical rule in `framework.md` §F3 (the product owner's decision, 14 Nov 2026), a day is complete only by the fact that all three meals are logged, or by the user's confirmation; the refresh writes totals only, and a day with missing meals stays open.
  - Re-runs update the one row (idempotent by date + idempotency key). The job never creates a second row for a date.
- **Silent / ping rules:** if the food source has nothing for today, do nothing and send nothing (the Ask Queue already asked). Message **only** if the save failed, or the day is not complete (meals missing) / flagged suspected-partial — one short line naming the issue, gently (which meal is missing, in the "What did you have for lunch?" tone). Otherwise silent.

## Job 4 — Steps sync (evening)

- **Consent key:** `steps_sync` — a separate consent row with its own user-chosen time, collected in onboarding Block 8g (distinct from the Block 8e source pick). `sourceKey` = `{{steps_source}}`, one of the enumerated steps keys (`health_connect` / `apple_health` / `manual` / `none`, `framework.md` §F2). If the user picked **manual** or **none** in onboarding, this job does not run: manual steps arrive when the user reports them; none means steps stay blank.
- **Time:** `{{steps_sync_time}}` (evening).
- **Trigger / inputs:** daily step counts for yesterday and today (`{{user_timezone}}` dates) from the chosen health source. If coverage for a day is incomplete or missing and a device backfill is available to the install, run the backfill for the missing range once, then re-query once.
- **Write:** for each day with a step count, upsert via the steps-log save: `{date, steps, source: <steps source key>, provStatus: observed, sourceRef: "daily steps sync"}` into `body_metrics`. The write preserves any weight already recorded for the same date.
- **Blank rule:** a day with no step data is left blank — never 0, never an estimate.
- **Scope limit:** this job touches step values only. It does not touch workouts, macros, baselines or any other record.
- **Silent / ping rules:** silent on success. On failure or block (for example the device is offline, or permission is gone), send ONE short note to `{{delivery_chat}}` saying steps sync is blocked and why, then stop.

## Job 5 — Morning brief (morning)

- **Consent key:** `morning_brief`. **Time:** `{{morning_brief_time}}` — in the morning, just after the user's own daily brief where they have one; the exact time is the user's choice from onboarding. **Delivery:** exactly ONE short message to `{{delivery_chat}}`.
- **Method — read fresh every run.** Gather:
  - the tracker: the Today dashboard read where present, otherwise composed from the macro summary, steps summary, weight summary, recommendations and overview reads. Targets, target weight and step goal are read from **this user's** records — the method contains no built-in target numbers.
  - `{{food_source}}`: yesterday's meal-by-meal detail — which meals are logged and how heavy the largest meal actually was. (Meal heaviness is a meal-level fact; the tracker's day totals cannot show it.)
  - today's training: sessions in the tracker for today, plus the user's calendar where the install has it — including any recurring session **this user** has (read from their records, never assumed).
  - steps and weight: as already synced into the tracker (Job 4 and weigh-ins). Do not re-query the device in the morning unless yesterday's steps are missing in the tracker.
- **Message structure, in order:**
  1. Greeting line.
  2. **Completeness chase (only if needed):** if any of yesterday's meals is still unlogged, ask for exactly those meals **by name, gently** — "What did you have for lunch?", never "Did you have lunch?" and never a generic "you didn't update". Ask once; only chase what is still missing after the evening Ask Queue; never repeat a question the user has already answered. The day stays open until the meals are logged or the user confirms it. If yesterday is fully logged, skip the chase entirely.
  3. **Three-point executive summary** — three points from the user's real numbers.
  4. **One numbers line** — yesterday and the 7-day picture (averages over complete days only, coverage as recorded).
  5. **ONE recommendation** — the closing line, chosen by the priority below, with its basis (the number that caused it) in the same breath. No basis, no recommendation.
- **Recommendation priority (pick ONE, highest first):**
  1. **Heavy largest meal last night** (that meal was the day's dominant meal, or the day ran well over target): steer light today for the first two meals, back on target by the last meal.
  2. **Protein pattern low** (target hit on under half of the last 7 complete days, or yesterday well short): anchor the first meal with protein today.
  3. **Training today:** fuel for it — carbs around the session, protein after. Name the session from the user's own records.
  4. **Rest day after an over-target stretch:** today is the pull-back day, stated plainly with the week's overshoot number.
  5. **Weight pace:** losing faster than about 0.75 kg/week → eat a bit more ("stronger, not just lighter"); or pace towards the target weight stretching far out → name it honestly with the estimate — only when at least 4 weigh-ins over at least 14 days support one.
  6. **Steps slipping** (below the user's step goal on 3 or more of the last 7 logged days): a movement nudge with the user's actual average.
  7. **Otherwise:** an honest on-track line. Never manufacture a problem.
  - Where the user has a goal mode, the steer follows that mode (Job 5 reads it from the goals record).
- **Format and tone:** short; greeting, chase if any, the 3 points as dash bullets, one numbers line, recommendation last. Plain, warm, direct — a coach who has read the user's data, not a report generator. Real numbers, real session names.
- **Personal context:** the summary and the steer reference the user's stated goals and what they want from the tracker (the goals record and the verbatim wants) **in the user's own words**, where the framework's data supports it — as personal context, never as generic motivational template text, and never invented when the records hold none.
- **Never:** generic health tips; invented foods the user did not eat; zero-filled gaps; averages over partial days presented as full days.
- **Guardrails:** if a source is unreachable, the brief says it is partial and names what is missing rather than guessing — above all, never guess what the user ate. Averages over complete days only; partial days labelled; blank stays blank. **This job never edits targets and never logs anything on its own.** It asks, summarises and recommends only.
- **Failure rule:** if the brief genuinely cannot be composed (all sources down), it sends one short line saying so rather than staying silent.

## Job 6 — Reconciliation (weekly)

- **Consent key:** `reconciliation`. **Time:** `{{reconciliation_time}}` (weekly, evening, user-chosen day/time).
- **Reference point creation (before the comparisons):** immediately before this job runs, the system takes a fresh backup snapshot under the creation procedure in `framework.md` (`createBackupSnapshot`: weekly before reconciliation, after any framework change, or on request; a manifest of per-table active/tombstoned counts, macro-log dates with revisions and completeness, target revision, session uid/date/status/revision list, and a checksum over the exported overview, stored append-only inside the install's own Space data). The comparisons below read the latest snapshot taken before the week under review.
- **Scope (the live scope for this product):** three read-only comparisons —
  1. the tracker overview + framework status counts against the install's **own backup/manifest**, adjusted for legitimate logging since the backup date (checked against the week's chat logging): the original baseline/plan/session/macro-day counts must still be present underneath new logging;
  2. the last 7 days of `{{food_source}}` day totals vs tracker macro logs: missing days, duplicate days, mismatched totals;
  3. the sync log for failed or refused syncs this week.
  - (An earlier plan-stage variant of this job also compared against a personal spreadsheet. That leg is **not** part of the product: the generic replica reconciles against its own backup/manifest and food source only.)
- **Output:** ONE short report to `{{delivery_chat}}`, headed "Reconciliation — [date]", containing either "all counts match, no issues" or a numbered list of the exact differences. Any day in the week with unlogged meals is flagged **not complete**, naming what is missing (for example "you didn't log lunch on Tuesday"); such a day is never silently treated as complete, and the week's averages follow the complete-days-only rule.
- **Rules:** always reports on schedule (this job's report is its delivery, unlike the silent-on-success jobs). **Fixes nothing.** The user decides what, if anything, to fix. A predecessor tracker that the user deliberately deleted or replaced is not flagged as "missing" — reconciliation compares against the install's own manifest, whatever it records.
- **On-demand surface (Gate G3 clarification P5):** the same report can be produced without the job firing, through the read action `getReconciliationReport(weekStart)` specified in `framework.md` — read-only, returning the same structure (duplicates, missing days, mismatched totals, failed syncs, not-complete days naming the missing meals), so the report is inspectable in any install, including one without a live cron.

## Sync health (cross-cutting behaviour — not a standalone job)

- There is **no standalone sync-health schedule** in this product. Sync health is a behaviour of the other jobs plus two in-app surfaces: the sync log (readable via the sync-log read) and the framework status read (active/tombstoned counts, history rows).
- Each job records its last success per source as it runs — concretely, the `lastSuccessAt` field (and the responsible `owner`) on the job's `automation_consents` record in `framework.md` (Gate G3 clarification P4), readable via `getFrameworkStatus` / the consent read. Failures are written to the sync log with source and reason.
- A source with no success for **48 hours**, where that source is expected to report, raises a stale warning to `{{delivery_chat}}`. Otherwise sync health is silent: it pings only on failure or staleness.

## Change recheck (event-triggered procedure — not a standing schedule)

- **Trigger:** any framework change to the install (schema, sync contract, mapping order, job definitions). Not a cron; an event-triggered procedure that runs **the morning after the change**.
- **What it does:** re-runs the acceptance pack relevant to the change (at minimum: the record-preservation diff against the manifest, and a check that every consented job still exists with its chosen time and owner).
- **Output:** a result-or-blocker report to `{{delivery_chat}}`. Nothing else: no report is sent when no change happened.
- This is the product form of a standing rule for new setups and fixes: after a change lands, recheck it the next morning and report the result or the blocker.
