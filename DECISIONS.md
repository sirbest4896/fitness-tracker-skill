# DECISIONS — BestFit

This log records how the open items from the Phase 0 reconciliation (Gate G0) were decided for this generic product.

**Status language — read this first.** Every item below is marked **PRD-DECIDED**: a design decision made for the generic product in this package, so the product is buildable and testable end-to-end. A PRD-decided item is **never** claimed as extracted from, or verified in, any existing user's live tracker. Where the underlying fact came from an audited source, the G0 resolutions already settled it in Phase 0 and it is not re-decided here; this log covers only what Phase 0 left open (O1–O7) plus the gaps this package closes by design.

Also recorded here, so they cannot be silently dropped: the Gate G0 gaps this package addresses by specification — G1 (More section), G4 (action catalogue), G5 (record_history / notes schemas), G7 (goals + consent data model), G10 (EWMA status), G13 (template 0-seed in the template spec).

---

## O1 — Step-factor table for maintenance — PRD-DECIDED

**Open item:** maintenance kcal = latest weight (kg) × a step-based activity factor, but no source supplied the table of step ranges → factor. Without it the goal-mode calculation could not run or be mock-tested end-to-end.

**Decision:** this product ships the following table, **PRD-decided v1, tunable**:

| Recent average daily steps | Factor (kcal per kg of body weight) |
|---|---|
| Under 5,000 steps/day | 31 |
| 5,000 – 7,999 steps/day | 33 |
| 8,000 – 11,999 steps/day | 35 |
| 12,000 steps/day or more | 37 |

- "Recent average" is defined normatively (Gate G1 closure B5): **the mean of the user's logged step days in the last 14 days**. Days with no step data are excluded from that average (never counted as 0-step days). **A minimum of 3 logged step days in that window is required**; with fewer, the step factor — and therefore maintenance — is unavailable and the missing input is named. There is no separate non-step "activity picture" input in this product; this step average is the only activity input.
- `maintenanceKcal = latestWeightKg × factor` for the band the average falls in.
- The table is a product parameter, labelled v1. An implementer may tune the bands/factors for the product as a whole; it must never be tuned per user from another user's data, and a change to the table is a framework change (it triggers the change-recheck procedure in `automation.md`).
- Where the user has no current weight, or the step average is unavailable under the window rule above, maintenance is unavailable: the calculation does not run, the missing input is named, and targets stay blank (see O3).

## O2 — Kcal-per-gram factors — PRD-DECIDED

**Open item:** the split maths ("carbs are the remainder") implies conversion factors that no audited source prints.

**Decision:** the product adopts the standard Atwater factors as design constants: **protein 4 kcal/g, carbs 4 kcal/g, fat 9 kcal/g**. Recorded here as an adopted standard, not as an audited fact from any existing tracker. Carbs are computed as: `carbsG = max(0, (targetKcal − proteinG×4 − fatG×9) / 4)`.

## O3 — Split preset/slider basis — PRD-DECIDED

**Open item:** the presets are expressed per kg of target weight; the onboarding script also offered a fallback to current weight that no source stated.

**Decision:** the basis is resolved in this order:

1. **Target weight**, if the user set one.
2. Else, the **latest current weight**, if one is logged.
3. Else, the calculation is **unavailable**: it does not run, the missing input is named, and targets stay blank.

The fallback in step 2 is adopted as a rule of this generic product (it originates in the package onboarding script, written for this product). It is not presented as extracted from any existing tracker. Whichever basis is used is recorded with the recommendation (`basisKind` = target / current, `framework.md`) and shown in the calculation's working.

## O4 — body_metrics population — PRD-DECIDED

**Open item:** the table's population path was unevidenced (a plan-stage intent to seed it from imported history sat next to a test-time count of 0 rows on a test duplicate).

**Decision:** in the generic product, `body_metrics` (date, weight, body fat, steps, water, sleep):

- **starts empty** in every install;
- is populated by exactly three paths: (a) the **steps sync** (steps, from the user's chosen source), (b) **manual weigh-ins** (weight, and body fat / water / sleep where the user supplies them), and (c) **manual step logs** (steps the user reports directly);
- is **never seeded from any spreadsheet** in the generic product, and never seeded from any other user's or install's history;
- follows the write-preservation rule: a steps write preserves the same date's weight and other metrics; a weight write preserves steps.

The plan-stage seeding intent is recorded as not adopted for the generic product. Nothing in this package claims any existing install's `body_metrics` contents.

## O5 — Home of the ambiguous-machine question, and pending confirmations generally — SUPERSEDED by the product owner's decision, 14 Nov 2026

**Open item:** the live Ask Queue runs 5 slots and does not include an ambiguous-machine slot, while an earlier plan list named one. The queue count itself was settled at Gate G0 (5 slots); only the machine question's home was open. Gate G0 (X3/O5) resolved the machine question out of the queue and into the in-chat Photo Rule flow; the Gate G1 fix pass then added a machine-only displacement exception.

**the product owner's decision (14 Nov 2026, in his own words — this supersedes G0 X3/O5 and the Gate G1 machine-only framing):** the rule is not about machines only. **Nothing is logged unless the user confirms it.** If the agent is unsure about anything (which machine is the main case), it confirms with the user in chat first ("Is this the one?"). If the user does not reply or forgets, the item stays unlogged and unlocked, and the pending confirmation goes into the nightly WhatsApp Ask Queue as its **last slot**, asking again. Still unconfirmed: still not logged — keep waiting.

**Decision as applied:**

- The Ask Queue keeps the **5 named slots** specified in `automation.md` (missing food; attendance; suspected-partial; blank weights; pain/readiness). There is no standing machine slot.
- The ambiguous-machine question **lives first in the in-chat logging flow**, under the Machine Photo Rule, at the moment of logging: stop, show the look-alike photos, ask in confirmation form ("Is this the one?"), log nothing until the user confirms. Under F5, the answer is bound to the draft record it clarifies; nothing else in the draft is blocked once it is resolved.
- **Pending-confirmation rule (generalises the machine question):** any item awaiting the user's confirmation — the queue item kind is `pending_confirmation`, with machine identity (`confirmationKind = 'machine_pick'`) the main case — that is still unanswered at night takes the **last position** in that night's queue, asking again. If all 5 slots are filled, it **replaces the lowest-priority present slot for that day only** — displacement runs in reverse priority order (slot 4 first, then slot 3, and so on), and slot 0 (missing food) is never displaced. With no pending confirmation, no such item appears. A pending confirmation never expires into a guess.

## O6 — Canonical provenance naming — PRD-DECIDED

**Open item:** two field names existed in the source material for provenance status, plus a third, user-facing vocabulary in onboarding ("user-given" / "calculated").

**Decision:**

- The canonical field is **`provStatus`**. It is the only provenance-status field name used anywhere in this package's data model, actions and job specs.
- Its values are exactly: **`observed` / `estimated` / `derived`**.
- Onboarding's user-facing words map to data values as follows, and this mapping is normative wherever the words appear:
  - "user-given" → `observed`
  - "calculated" → `derived`
  - "estimate" (a food estimate) → `estimated`
- User-facing copy keeps the plain words (user-given / calculated / estimate); stored data always uses `provStatus` and the three enum values. No other provenance field name or value set may be introduced by an implementer.

## O7 — Booking end-to-end path — PRD-DECIDED (PRD-specified)

**Open item:** two evidenced halves did not join up: an email-driven mirror into a user's main calendar (live behaviour in one install), and calendar/email arrivals becoming scheduled-only sessions in the tracker calendar under F1 (specified in the plan and the skill behaviour). The joining step was unevidenced.

**Decision:** this product **specifies** the end-to-end path as follows, and the PRD and `automation.md` state it in these terms:

> **Source (calendar or email, each separately consented) → booking mirror job → tracker calendar, as a scheduled-only session via the F1 contract → the user's attendance confirmation converts it to completed.**

- The mirror writes through `upsertSyncedSession` under F1: match order externalRef → uid → fuzzy key, idempotency key from the source record, source-owned fields only, tombstone on cancellation, move-not-duplicate on reschedule.
- Attendance confirmation is the only path from scheduled to completed (completed status is user-owned).
- This path is marked **PRD-specified**: it is a specification decision for the generic product, consistent with all evidenced fragments (scheduled-only arrivals, the F1 contract, the mirror pattern, the attendance gate). It is not a claim about the internal wiring of any particular existing install.

---

## Gaps closed by design in this package

Each item below is a **PRD design decision** for the generic product, made so the package is buildable. None is claimed as extracted from a live install.

- **G1 — More section.** Evidenced only as the fifth section. Decided: More is an implementation-defined minimal settings/help area, specified in `PRD.md` FR-MORE-2 and `design-specs.md` §8 (profile basics, goals and target state, automation consent list, notes, short help text). Nothing beyond that minimum is required or claimed.
- **G4 — Action catalogue.** The evidenced actions are listed as evidenced in `framework.md`. The additional CRUD/settings actions a builder needs (baselines, plans, body-metric writes, muscle override, Apply, goals, consents, profile, ambiguity resolution) are specified there as **PRD-designed actions**, all bound by F1/F5.
- **G5 — record_history and notes schemas.** Column sets are specified in `framework.md` and labelled PRD-designed.
- **G7 — Goals and consent data model.** The `goals`, `goal_recommendation_state`, `automation_consents` and `user_profile` records are designed in full in `framework.md` and marked PRD-designed. They hold exactly what `onboarding.md` collects: goal in own words, goal mode, target weight, pace, step goal, 12-week success, estimates permission, and a separate consent row per automation with its user-chosen time and the do-not-send rule.
- **G10 — EWMA status.** No source confirms what trend method, if any, an existing chart uses. Decided for this product: the weight trend is **EWMA-lite** — exponential smoothing with **alpha 0.3** over logged weigh-ins, computed over full logged history, displayed for the recent window, with the **line breaking at gaps** in logging (no interpolation across a gap). Fewer than two weigh-ins → no trend line. The trend is a derived display value (`provStatus = derived`); raw weigh-ins are never overwritten by it. If a future version implements a fuller time-aware EWMA (daily grid, a stated half-life), that is a framework change and follows the change-recheck procedure.
- **G13 — Template 0-seed.** The template spec in `SKILL.md` carries the 0 rule in all three contexts: template seed 0 = "not set yet" placeholder; all logic treats 0 exactly like blank; an unknown weight in a logged session is blank, never 0. Onboarding's user-level phrase "starts completely empty" refers to the user's data, not to the catalogue seed, and the two statements are consistent under this rule.

## Goal formulas — summary table (Gate G1 closure F4)

The normative statements live in `framework.md` §Goals and `PRD.md` FR-GOAL-*; this summary exists so the pointer in `SKILL.md` ("defined in `framework.md` and `DECISIONS.md`") is true in both places. Values are digit-identical to those files:

- **Mode multipliers on maintenance:** Lose Fat ×0.85; Recomposition ×0.97; Gain Muscle ×1.07; Gain Weight ×1.12.
- **Split presets (protein g/kg, fat g/kg, on the O3 basis weight):** Balanced 2.1 / 0.95; High Protein 2.8 / 0.85; High Carb 1.9 / 0.7; Low Carb 2.2 / 1.4. Protein and fat are slider-adjustable from the preset.
- **Kcal factors (O2):** protein 4 kcal/g, carbs 4 kcal/g, fat 9 kcal/g. Carbs are always the remainder: `carbsG = max(0, (targetKcal − proteinG×4 − fatG×9) / 4)`.

## Gate G1 amendments and buildability closures (PRD-designed)

Recorded at the Gate G1 fix pass, 14 Nov 2026. All items below are design decisions for the generic product, in the same status language as the rest of this log:

- **Completeness — Gate G1 derived-completeness amendment WITHDRAWN, replaced by the product owner's decision (14 Nov 2026, below).** The Gate G1 fix pass had allowed the food-source refresh to set a macro day `complete` as a derived value; the Gate G1 reconciler ruled that this superseded G0 A4 ("complete is user-asserted") without authority. The product owner then decided the point himself (next section). The derived-completeness model no longer appears anywhere in this package: the refresh never writes completeness.
- **Onboarding amendment (amends `onboarding.md` Blocks 8 and 9).** The onboarding script is **amended** at this gate — this log is the amendment record. Block 8 now collects all seven automation consents: the original five items (8a Ask Queue, 8b morning brief, 8c calendar booking, 8d email booking, 8e steps source) are unchanged in wording and order, and three items are added in the same style — **8f macro refresh** (separate consent + its own time + the food-source binding, from the enumerated food keys), **8g steps sync** (separate consent + its own time, distinct from the 8e source pick), **8h reconciliation** (separate consent + its own weekly day/time) — plus a booking-mirror interval confirmation (template default 30 minutes) asked when either booking sync is on. Block 9's automation read-back now lists all seven, the interval and the food source. Marlowe's style is kept throughout, and the exact save wording in Block 9 is **unchanged**.
- **B4 — Backup/manifest creation.** Specified in `framework.md` (`createBackupSnapshot`: weekly before reconciliation, after any framework change, on request; manifest contents enumerated; stored append-only inside the install's own Space data).
- **B5 — Step-factor window.** Defined in O1 above and in `framework.md` §Goals: mean of logged step days in the last 14 days, minimum 3 logged days, else unavailable; no separate non-step activity input exists.
- **B6 — Source keys.** Enumerated in `framework.md` §F2: food keys `food_tracker` / `manual` / `none` (bound at onboarding Block 8f); steps keys `health_connect` / `apple_health` / `manual` / `none` (picked at Block 8e). Writes with an unlisted key are refused.
- **B7 — Booking-mirror interval.** A template parameter (`{{booking_mirror_interval}}` / `intervalMinutes`): generic default **30 minutes**, recorded only as confirmed or changed by the user at onboarding.
- **B8 — Training-day completeness criteria.** Stated in `framework.md` §F3: complete = an attendance outcome confirmed for every session scheduled that day, or a completed session logged; partial = some but not all; none = nothing.
- **B9 — `getMorningBrief` contract.** Inputs/outputs shape specified in `framework.md` (Actions); the morning-brief job composes from it (or from the dashboard/summary reads it wraps). It writes nothing.
- **B10 — `getAskQueue` item shape.** Specified in `framework.md` (Actions), including the per-slot fields and the `pendingDraftUid` / `confirmationKind` / `photoPairKey` fields the pending-confirmation rule (O5, as decided by the product owner) depends on.

## the product owner's decisions, 14 Nov 2026 (supersede Gate G0 on both points)

Recorded from the product owner's own words on the evening of 14 Nov 2026, given in response to the Gate G1 reconciliation FAIL. (The owner is not named in this package, under the package's privacy rule; the decisions are recorded verbatim in substance.) These are the product owner's explicit decisions; they supersede the G0 resolutions named below and are the authority the Gate G1 reconciler's route (b) required.

- **Decision 1 — Completeness (supersedes G0 A4).** A food day is complete **only** when breakfast, lunch and dinner are all logged, or the user confirms the day (for example confirming a meal was skipped, or that the day is done). When all three meals are logged, the day is complete **by that fact** — no job declares it. There is **no derived auto-complete**: the macro refresh may compute and show that all three meals are present, but completeness changes only from logged meals or user confirmation, and the refresh never writes it. The system **never closes or locks a day on its own**: a day with missing meals stays **open**, and missing meals are asked about **gently** — "What did you have for lunch?", not the rigid "Did you have lunch?" — ask and wait, and ask again the next day if still unanswered. In a weekly summary, a day with unlogged meals is flagged **not complete**, naming what is missing ("you didn't log lunch on Tuesday" style), never silently treated as complete; averages follow the complete-days-only rule. Training-day completeness is unchanged (user-asserted, criteria in `framework.md` §F3). The canonical paragraph in `framework.md` §F3 is normative; `PRD.md`, `guardrails.md` and `automation.md` state the same rule in substance.
- **Decision 2 — Unconfirmed items (supersedes G0 X3/O5).** Nothing is logged unless the user confirms it. If the agent is unsure (which machine is the main case), it asks in chat first, in confirmation form ("Is this the one?"). If the user does not reply or forgets, the item stays **unlogged and unlocked**, and the pending confirmation is carried into the nightly WhatsApp Ask Queue as its **last slot**, asking again. Still unconfirmed after that: still not logged — keep waiting; it never expires into a guess. The full rule is recorded at O5 above.
- **Application note:** both decisions were applied across `framework.md` (§F3 canonical paragraph, §F5, `getAskQueue`), `PRD.md` (FR-MAC-4, FR-FW-3, FR-AUTO-3, AC-16/28/32/37/40, N19–N24), `automation.md` (Jobs 2, 3, 5, 6), `SKILL.md`, `guardrails.md` (rules 24–25) and `design-specs.md` (§5) in the fix-g1c pass; the `onboarding.md` closing explainer, which stated the old partial-day wording, was aligned to the two decisions in the same pass (the exact save wording was not touched).

## Gate G2 amendment — design identity and build rules pinned (14 Nov 2026)

Recorded at the Gate G2 fix pass, 14 Nov 2026. The G2 visual checker found that the package had not pinned the product's design identity, so the replica builder invented one (a different accent, typeface, background and navigation placement). The following are now specified in `design-specs.md` as the product's design identity and build rules. These are design facts of the product, not any user's personal data:

- **Design identity (design-specs §1):** accent volt lime `#B8E62E`; background warm off-white `#F4F2EC`; headings Barlow Condensed with Barlow body type; **bottom navigation** with sub-labels (not a top pill bar); macro colour coding protein teal `#14919B` / carbs amber `#E9A23B` / fat blue `#3A6EA5` applied to the target, breakdown and charts everywhere macros appear, including the Macros page target/breakdown (Gate G2 visual item 4).
- **Chart windows (design-specs §2):** the Today stacked-bar macros chart and steps bar chart span **the last 7 days** (the weight line stays 30 days).
- **Macros page (design-specs §5, reaffirmed):** the past-7-days chart, the colour-coded target/breakdown, and **separate** Steps and Weight blocks are required components, not optional.
- **Photo coverage (design-specs §7):** every catalogue exercise carries a verified real photo sourced at build/install time from generic external sources and stored in the Space; a photo-less catalogue exercise is a build defect. The package `photos/` set remains the mandatory look-alike disambiguation set and is never replaced by build-time sourcing; no personal image is ever used.
- **Demo furniture:** test scaffolding (for example a "try the sync path" demo form on Calendar) is not product design and must not appear in a shipped build; the booking-mirror behaviour lives in the sync actions, not in a demo form.

## Product owner's design decision, 14 Nov 2026 — theme and body figure (supersedes the Gate G2 theme)

Recorded from the product owner's own words on 14 Nov 2026, after he reviewed phone captures of the replica built to the Gate G2 amendment above. He judged the heat-map figure's shape unacceptable (in his words, the person in the image looked "mocked as so fat") and the overall look "a bit ugly", and set the colour themes himself: **dark blue, black, and red — those three.** (The owner is not named in this package, under the package's privacy rule.) This is the product owner's explicit design decision; it supersedes the theme portion of the Gate G2 amendment above. That amendment's other rulings (chart windows, Macros page components, photo coverage, no demo furniture) are unaffected and stand.

- **Theme (design-specs §1, normative):** background near-black `#04060B`; surfaces deep navy `#0B1526` (raised `#111F36`, borders `#22354F`, nav bar `#081120`); dark blue primary `#1B4B8F` (active state `#2F6FD0`); red accent `#FF3B47` for highlights, the active tab, key numbers and calls to action; text `#F2F5FA` / `#9AA9C2`. Barlow Condensed / Barlow type and bottom navigation are unchanged.
- **Scope of the supersession:** the Gate G2 amendment had pinned the replica to the live tracker's theme (volt lime on warm off-white) for 1:1 parity. The owner has replaced that parity theme **for the replica product**: the skill now specifies the dark blue / black / red theme above. The live tracker keeps its own theme; nothing in this decision changes it, and no function, data rule or behaviour changes with the theme — chrome only.
- **Data colours retained, deliberately:** the muscle-state palette (Sore `#C8102E` / Recovering amber / Ready green) and the macro coding (protein teal `#14919B` / carbs amber `#E9A23B` / fat blue `#3A6EA5`) are functional data colours, not theme, and stay — changing them would break at-a-glance reading of states. The separation rule is specified in design-specs §1: the red accent is chrome only and never appears inside the body figures; a muscle is red only as a labelled Sore gradient fill.
- **Body figure (design-specs §6, normative):** the front and back figures are redrawn as a **lean, athletic silhouette** — defined shoulders, tapered waist (shoulder-to-waist ratio roughly 1.7:1), head-to-body about 1:7, slim straight legs with a visible gap — explicitly not a wide, rounded or stocky figure. The 18 muscles, soft gradient zones, labels beneath, and the adductors hard rule (two slim inner-thigh strips, never one central blob) are unchanged.
- **Application note:** applied in the theme-amendment pass to `design-specs.md` (§1 theme + data-colour separation, §6 silhouette) and `PRD.md` (AC-43 restated to the new theme; new AC-52 / test N34 for the silhouette). No build was changed in this pass; the replica is re-themed only in a later build run, after a checker verifies this amendment (the owner's PRD-before-build rule).

## Product owner's name decision, 14 Nov 2026 — the product is called "BestFit"

Recorded from the product owner's own words on 14 Nov 2026: **"Let's call it best fit"** — chosen because fitness covers training + macros/food + steps, which is exactly what this one framework holds. (The owner is not named in this package, under the package's privacy rule.)

- The display name **BestFit** replaces "Fitness Tracker 2.0 Replica" / "Fitness Tracker Replica" in the package's titles and product references (`SKILL.md`, `PRD.md`, `design-specs.md`, `framework.md`, `automation.md`, `guardrails.md`, `onboarding.md`, this log's title).
- The skill's machine id is **unchanged**: `fitness_tracker_2_replica`. Renaming an id risks breaking installs and references; the name change is a display/product name change. Dated entries elsewhere in this log that use the former name are historical records of what the product was called at that gate and are not rewritten.
- The replica Space's **title** takes the new name at build time (a later build run, per the owner's PRD-before-build rule). No file outside this package is renamed by this decision; any existing live tracker keeps its own name.

## Product owner's new-user-experience decision, 14 Nov 2026 (late evening) — framework framing, onboarding order, connectors, packaged prompt, personal onboarding

Recorded from the product owner's direction on the evening of 14 Nov 2026. (The owner is not named in this package, under the package's privacy rule.) This decision governs the new-user experience only; it does not touch the two decisions of the same evening recorded above (completeness; unconfirmed items), the exact save wording, or the seven per-job consents — all stand unchanged in substance.

- **Framework framing.** The product is a framework from the start, not a fixed app. New users are told early and plainly — in a charter stated at the start of onboarding — that they can **always customise everything**: theme/colours, targets, goal mode, exercise catalogue, automation jobs and their times, meal structure, and section behaviour where the framework allows. Nothing is locked; every default is a starting point, changeable through the agent at any time, not just during onboarding.
- **Onboarding order.** The onboarding narrative is restructured to this sequence: **(a) the automation first** — the cron jobs explained first (what each does, when it runs, what it sends), each separately opt-in with its own time, all default off (the existing Block 8 material, run as Stage A; the timezone is asked there because every time in that stage is in it, and Block 1 then confirms it); **(b) connectors** — the user is told what to connect, each connector named with its one-line unlock (food source → macro refresh; health platform → steps sync; calendar → booking mirror as scheduled sessions; email inbox → booking-confirmation path), each optional, none assumed, choices recorded per `framework.md` §F2 source keys, and any Stage A consent a connector answer changes is updated and shown in the read-back; **(c) then everything** — basics and training life (including the days that fit the user's week), the open question **"What do you want from this tracker?"** (answer stored verbatim as `trackerWants`; mapped to features out loud; out-of-scope wants stated honestly at onboarding, never promised), the existing Goals block and the remaining blocks; **(d) the capability tour** — what the tracker is capable of, in plain language, before the read-back. All nine blocks' content and rules are kept; the amendment reorders the narrative and adds the charter, Stage B, Stage C and Stage D material (`onboarding.md`).
- **Packaged onboarding prompt.** A new package file, **`onboarding-prompt.md`**, ships with the skill: a ready-to-use, copy-paste prompt the new user personalises **themselves** — with fill-in sections including **their meals** (which meals make up their day, or their own pattern; rough times; how they report food; their food source; estimates permission), their goals in their own words, what they want from the tracker, training setup, connector choices, automation choices and their own times, and theme choice (default: the pinned dark blue / black / red identity, changeable). It states the framework framing: their answers personalise it; blanks stay blank and are asked about, never filled by assumption. `onboarding.md` references it as the packaged companion (a filled-in prompt is confirmed part by part, not re-asked from zero).
- **Onboarding must be personal.** The owner stressed this twice ("the onboarding is super important"; "make it personal"). Onboarding speaks to the user as a person, not a form: their name used naturally once known; their goals, wants and success picture played back **in their own words** (at the read-back, and later — the morning brief, Ask Queue and Today steer reference their stated goals/wants as personal context where the framework supports it, never as generic templates or invented motivation); questions shaped around their meals, their routine, their life pattern (Block 2 now asks which days fit their week); their pace and comfort respected (one topic at a time, no interrogation feel, pause/resume offered, skipped stays blank and is asked gently later). Personalisation never fills a blank: personal context comes only from what the user actually gave.
- **Meal structure note (for the checker).** The user's own meal pattern is recorded (`user_profile.mealStructure`, `framework.md`) and is the set the gentle missing-meal asks and the day view use — a user is only ever asked about meals they actually eat. The completeness rule itself is **unchanged in substance** (Decision 1, above): complete only when breakfast, lunch and dinner are all logged, or the user confirms the day; never written by a job; never auto-closed. `mealStructure` governs the gentle asks and the day view only — it never changes the completeness test; a user whose pattern differs confirms the meals they don't eat, and that confirmation is what completes the day.
- **Application note:** applied in the NUX amendment pass to `onboarding.md` (restructured; charter, Stages A–D, personal rules, Block 1 timezone-confirmation, Block 2 days question, Block 7/8 transition lines aligned to the new order, Block 9 read-back extended with wants + connectors lines, checklist extended), new `onboarding-prompt.md`, `PRD.md` (§1 framework framing, US-17–US-20, FR-ONB-1 order wording, new §4.11 FR-NUX-1–FR-NUX-7, AC-53–AC-59, tests N35–N41), `framework.md` (`goals.trackerWants`; `user_profile.trainingDaysFit` / `mealStructure` / `connectorChoices`), `automation.md` (onboarding-position note; timezone placeholder wording), `SKILL.md` (name, charter, onboarding section in the experience order, deliverables). Goal formulas were not touched and remain digit-identical. No build was changed in this pass; replica changes follow in a later build run after a checker verifies this amendment (the owner's PRD-before-build rule).

## Gate G3 reconciliation clarifications, 14 Nov 2026 — framework completion, NOT new owner decisions

Recorded from the Gate G3 reconciliation (`gate-g3-reconciliation.md`). These are **framework clarifications derived from sources already in this package and the owner's standing framework** — they complete underspecified points; none of them is a new decision by the product owner, and none changes any owner decision recorded above. Where a question looked open (restore semantics), the reconciliation settled it from the owner's own framework sources: AC-3's expected history (created → moved → cancelled → **restored**), §F1's "a supported restore path returns the record to active", and the `(source, externalRef)` uniqueness rule all point the same way.

- **P1 — Suspected-partial quantified (§F3).** The complete-day median is the median of daily calorie totals over complete days in the last 28 days, requiring at least 3 complete days (otherwise the calorie signal is unavailable, never guessed). A partial day flags when at least one meal is logged and at least one is missing **and** its total is below 50% of that median, **or** the only logged meal is breakfast. Evaluated on every macro-log write for the date and recomputed at Ask Queue composition; the writer is the macro-log write path / queue read; the macro refresh never writes the flag; the reason is stated in plain words; flagging never writes completeness. Mirrored in PRD FR-MAC-4; tested by AC-60 / N42.
- **P2 — Restore semantics (§F1).** A genuine re-booking restores the tombstoned record: trigger = `cancelled: false` + changed date/time + (where supplied) a source timestamp later than the cancellation event; outcome `restored`, same `uid`, `deletedAt` cleared, `revision` incremented, history transition `cancelled → restored`. An identical or earlier payload is stale: outcome `tombstoned-kept`, no change. Two surfaces: the sync-side restore inside `upsertSyncedSession`, and a user-confirmed `restoreFromSnapshot(snapshotId)` (F5 applies; idempotent replay/read-back into a scratch state) — the surface AC-18 / D8 is verified through. AC-3 states the trigger; the Stage D capability-tour claim about restore is **correct and unchanged**. Tested by AC-61 / N43 and AC-62 / N44.
- **P3 — Training-day completeness exposure.** The Train data read returns per-day training completeness (date, state, basis in plain words) for every day with scheduled or logged sessions. The field's home is a **per-day aggregate**, not a per-session column. Criteria unchanged (§F3 / B8). Tested by AC-63 / N45.
- **P4 — Job record surface.** The `automation_consents` / job record carries `owner` (the agent or user role responsible) and `lastSuccessAt` (per job, and per source where source-bound; null before the first success), exposed via `getFrameworkStatus` / the consent read. This names the field and surface AC-17 / D7 always required. Tested by AC-64 / N46.
- **P5 — Reconciliation report surface.** `getReconciliationReport(weekStart)` is a PRD-designed read action returning the Job 6 report structure (duplicates, missing days, mismatched totals, failed syncs, not-complete days naming the missing meals); read-only, changes nothing. Tested by AC-65 / N47.
- **P6 — Observation promotions and known behaviours.** Promoted to specification: macro-log reads surface `provStatus`, `completenessProvStatus`, `source`, `sourceRef`, `revision` (O-a); `getTodayDashboard` answers as-of the requested date (O-c); backup-manifest session entries state the record's true status including tombstone (O-g); `provStatus` transitions are written as `record_history` rows (O-j — stated explicitly in §F2). Recorded as **known behaviour, no change**: the Ask Queue surfaces only the oldest pending confirmation, one per night (O-b — now stated in the `getAskQueue` contract); muscle states are maintained at write time, not re-derived on read (O-d — stated in §F4); the estimates-permission gate is enforced in the chat flow, not the app layer (O-e — cross-referenced at guardrail 10). Rounding order for the goal calculation is now stated (O-h): protein and fat rounded to whole grams first, carbs computed from the rounded values (PRD FR-GOAL-4; `framework.md` Goal-mode maths). Tested by AC-66 / N48.

## Items deliberately NOT decided here

- **Per-muscle set targets** (Gate G0 gap G14): recorded as unimplemented in the source product and out of scope in `PRD.md`. No decision is taken to add them.
- **A shipped catalogue size counted in exercises** (Gate G0 gap G12): the package ships the named starting catalogue in `catalog.md` and the per-exercise verified-photo rule. No grown install's library count is adopted as a requirement.
- **Any existing install's configuration** — its job times, delivery channel, sources, owner bindings, or stored values — is not a decision input to this log and appears nowhere in the package as a default.
