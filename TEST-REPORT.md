# BestFit — Test Report (verification record for this skill package)

**What this file is.** The verification story of the BestFit skill (Fitness Tracker Replica Skill v3): how the package was checked, what the from-package replica build was tested against, what failed, what was fixed, and what was re-tested. Every test ran against **mock data only**, in a disposable test replica, against two fictional test personas ("Alex Tan", the canonical seeded user, and "Riley Mock", used once for the onboarding replay). No real user's data was used at any point, and no real tracker was ever written to by a test.

**How to read the verdicts.** The standard applied throughout, set by the product owner: *the skill is only complete when a replica built from the package alone passes.* A test that could not be run because the package gave it no surface was recorded as not-run, never as a pass. Checkers were independent of makers at every gate, and gate reconcilers did no fixing.

**Honest status up front.** Everything in the functional and framework test programme below passes. **One evidence item remains outstanding: Gate G2 item 6** — a rendered screenshot of the exercise **library grid** itself (see §3). The library's data-level coverage is certified (every catalogue exercise carries a verified photo; the photo card component is pixel-verified on the recommended-workout surface), but the library grid has never been photographed on a device, so that one visual item is recorded as **NOT EVIDENCED**, not passed. This report does not claim total completion while that item stands open.

---

## 1. Gate G0 — Phase 0 reconciliation (audit → package inputs)

Three independent Phase 0 deliverables (a full spec audit, an independent checklist, and a generic onboarding draft) were reconciled by a fourth, independent checker against the primary sources.

- 13 load-bearing agreements reached independently from different source sets, including: the F1 sync contract as the data layer; field ownership (sync never overwrites user-owned fields); completeness as three states per day; the exercise→muscle mapping order (specific names checked before generic ones); the exact confirm-before-save question — *"Ready for me to save this? Reply yes to save, or tell me what to change."*; heat-map cooldown values; and the rule that one user's personal constants must be parameterised, never shipped as defaults.
- Conflicts (including live-instance cron times vs documentation approximations, and a six-items-under-a-"max 5" heading contradiction in the source plan) were settled from primary sources, with instance values recorded as instance configuration, not generic requirements.
- Open items O1–O7 were carried into the PRD as explicit decisions rather than left implicit.

**G0 verdict: reconciled; Phase 1 (package authoring) authorised.**

## 2. Gate G1 — package consistency, and the two owner decisions

The first G1 check failed on fixable items; a fix round followed. The G1 gate reconciliation then **failed on two blockers**: the fix round had departed from the G0 agreement on two points — (1) completeness had been re-modelled as a system-derived value, and (2) the Ask Queue's unconfirmed-item slot had been narrowed to machine picks only. Each blocker was clearable only by reverting, or by an explicit decision from the product owner recorded in `DECISIONS.md`.

The owner decided both points (14 Nov 2026), and the package was brought to his rules:

1. **Completeness (Decision 1, supersedes G0 A4).** A food day is complete **only** when breakfast, lunch and dinner are all logged, or the user confirms the day. No job or process ever writes completeness; the system never closes or locks a day; incomplete days stay open; missing meals are asked for gently, by name ("What did you have for lunch?"); weekly summaries flag unlogged-meal days as not complete, naming the missing meal. The derived-completeness model was withdrawn in the record and removed from the schema (`completenessProvStatus` takes the single value `observed`).
2. **Unconfirmed items (Decision 2, supersedes G0 X3/O5).** Nothing is logged unless the user confirms it. Unsure items are asked in chat first ("Is this the one?"); with no reply, the item stays unlogged and unlocked and is carried into the nightly Ask Queue in the **last position** as a `pending_confirmation`, displacing only the lowest-priority present slot (slot 0, missing food, is never displaced). Still unconfirmed means still not logged — it never expires into a guess.

A third check (G1c) passed, and the final G1 reconciliation verified from the normative texts — not from the fix report — that both blockers were cleared by the recorded-decision route, that the withdrawn models left no surviving normative statement, and that the privacy grep was clean.

**G1 verdict: PASS. Phase 2 (replica build from the package alone) authorised.**

## 3. Gate G2 — replica parity: function PASS; visual PASS except item 6

The replica was built **from the package alone** (the builder was forbidden from consulting any live tracker), seeded with the fictional Alex Tan only.

### G2-A — function and data

The first function check failed on six defects (D1–D6): a duplicated Ask Queue slot under displacement; Today steps/weight targets reading null; the weight 7-day average showing the latest value instead of the mean; week-on-week covering fewer than its full measure set; the recommended-workout title not following the muscle mix; and blank baselines reading back as "0". All six were fixed and then **independently reproduced as fixed** in a re-check:

- Queue slots unique; the pending confirmation holds the last position.
- Targets read from the goals record (steps 9,000; weight 74.0 in the mock).
- Weight 7-day average = the mean (75.9 / 75.7 / 75.5 → **75.7**).
- Week-on-week carries all 7 rows with coverage labels on both sides.
- A 3 Leg + 3 Push mix titles the recommendation "Full-body".
- Every baseline's weight label is a real weight or null — never "0".

A forensic ruling in the same re-check confirmed the owner's completeness rule held in the build: a mock day that read complete with a meal missing traced, through the field-level history, to an explicit user-confirmation write — a preceding totals-only rewrite had left the day partial. The personal-data sweep of the replica passed unconditionally.

**G2-A verdict: PASS.**

### G2-B — visual

The first visual check failed (a missing past-7-days Macros chart, invented theme elements, an invented Calendar form, wrong chart spans, photo-less library cards). Per the owner's PRD-before-build order, the package was amended first (design identity pinned; Macros/photo/chart-span rules; new acceptance criteria), the amendment was checker-passed, the build followed, and the function re-check passed again. Pixel evidence was then supplied as phone captures by the owner, and a visual evidence checker ruled:

- **Macros page: PASS** — stacked 7-day chart in the macro colours, gaps for unlogged days, faded partial days, the coverage text verbatim, and the colour-coded daily target; the separate Steps/Weight blocks were verified by source and payload reads (recorded openly as source/payload-verified rather than pixel-verified).
- **Heat map: PASS** — soft radial gradients; adductors render as **two slim inner-thigh strips**, never one centre shape (verified on a 3× zoom crop).

The product owner then **changed the product theme** and the figure: dark blue / black / red chrome, and a lean athletic silhouette (his decision, recorded in `DECISIONS.md`; it supersedes only the earlier theme portion of the spec). The package was amended first, checker-passed, then built. On the owner's fresh captures of the re-themed build, the evidence checker ruled:

- **Theme: PASS** — near-black background, deep navy cards, dark-blue primary actions, red reserved for the active navigation tab and key highlights; no trace of the superseded theme; data colours (macro teal/amber/blue; muscle crimson/amber/green) unchanged, with the chrome red never used as a data fill.
- **Lean silhouette: PASS** — shoulders widest, tapered waist, slim straight legs with a visible gap, all zones inside the outline, front and back; adductors again verified as two slim strips on a zoomed crop.
- **No regressions** — Macros, Today, Calendar and the pending-confirmation card all render as previously passed.

- **Item 6 — exercise library grid: NOT EVIDENCED (still outstanding).** No capture in either evidence round shows the library catalogue grid itself. The recommended-workout photo cards are pixel-verified (real photos, correct card format, no overflow), which proves the photo pipeline and the card component; the package's design spec names the library and the recommendation grid as different surfaces, and the checkers declined to pass one surface on another's evidence. Data-level coverage stands separately: every catalogue exercise carries a verified photo (43/43 baselines photographed in the replica; the package `photos/` set covers all look-alike pairs). **What closes this item: one phone capture of the Train tab scrolled to the exercise library grid.**

**G2 position: function PASS; visual PASS on every item except item 6, which awaits that single screenshot. No code change is indicated by any visual evidence taken.**

## 4. NUX amendment — check cycle

A late amendment (owner-directed) restructured the new-user experience: framework/customisation framing first, automation consent first in the onboarding order, a connectors step, the goals block plus the open "what do you want from this tracker?" capture, a capability tour, a personal (non-form) onboarding tone, the packaged `onboarding-prompt.md` for users to personalise themselves, and the product name **BestFit**.

The first NUX check failed on two substance points: (F1) the packaged prompt's meals section had redefined completeness against the user's own meal list — contradicting Decision 1 — and (F5/F3) the personal-playback requirement and the prompt's rule statements were not fully specified or testable. A fix round corrected the prompt wording (the meal list governs what the tracker *asks about*; completeness stays the canonical rule), aligned the capability-tour line, pinned the framework and DECISIONS notes, and added testable criteria (AC-58's text-consistency check; AC-59 clause (f) covering morning-brief and Ask Queue playback). The scoped re-check verified the corrected wording verbatim, swept the package for weakening phrasings with no survivors, and confirmed the standing invariants: the exact save question byte-identical everywhere it appears; all 7 automation consents intact and separately answered; goal formulas digit-identical across PRD, framework and DECISIONS; no clock-time defaults anywhere (all times are user fields); privacy grep zero.

**NUX verdict: PASS; cleared for the replica build.** The rebuilt replica's onboarding surfaces (charter, staged order, connectors, wants capture, tour, personal playback) were verified in the build run and again inside Phase 3 (N35–N41).

## 5. Phase 3 — the mock-data test suite

The full suite (59 tests: T1–T10, D1–D8, N1–N41, against acceptance criteria AC-1…AC-59) was run live against the replica through its published actions. All write probes used reserved future test windows and clearly-prefixed mock fixtures.

**Tally: 41 PASS · 3 FAIL · 4 covered by prior gate evidence · 5 NOT-RUN in full · 3 NOT-RUN in part · 10 observations recorded.**

Passing highlights, all reproduced live: sync idempotency and replay no-ops (T1); reschedule matching, move-not-duplicate, and ambiguity handling (T2); refresh/correction discipline with field-level history (T4, T6); averages over complete days only, with the coverage denominator printed (T5, D1); exercise→muscle mapping order (T7, T8); field ownership — re-sync never overwrites a completed session's logged work (T9); preservation diff with zero silent losses (T10); same-day multiples (D2 — a diagnostic never previously run); duplicate detection across different idempotency keys and rejection of a logged weight of 0 (D3); never-logged muscles reading as no-data, with manual overrides stored distinctly (D5); hand-computed goal-mode maths for all four modes × all presets, including the carbs clamp at 0, with the stored target immovable except through Apply + Confirm (N3, N5); the EWMA-lite weight trend matching hand computation point-by-point; Ask Queue slot order, gentle by-name wording and the contracted payload shape (N22); the morning brief's chase behaviour and read-only reads (N13); unlisted food sources refused and logged (N23); and the NUX tests N35–N38, N40, N41.

**The 3 failures — two root causes:**

- **F1 (D6 / AC-16):** the suspected-partial flag never fired on a fresh qualifying write. Queue mechanics were correct on a pre-flagged seeded day, but the package specified the flag's *signals* in words only — no threshold, no median window or minimum sample, no evaluation timing, no named writer — so the behaviour was not deterministically buildable.
- **F2 (T3 / AC-3; second face N39 / AC-57):** no restore path existed for a tombstoned synced record. A genuine re-booking of a cancelled booking returned `tombstoned-kept`, and no restore action was published — which also made one capability-tour claim ("a restore brings records back with their history") untrue of the build.

**Not-run items:** D7 (job runtime legs — the replica installs no live jobs by design), D8's restore leg (blocked by F2), N1 (a fresh onboarding run — no reset surface), N14 (no reconciliation read surface), N20 (training-day completeness specified but exposed by no read), and partial legs of N4/N6/N7 plus the EWMA fewer-than-2-weigh-ins edge (empty/near-empty states unreachable on the seeded install with the published actions).

## 6. Gate G3 — reconciliation rulings (P1–P6)

An independent gate reconciler ruled on every failure, not-run item and observation. No new owner decision was needed: the one question that looked open — restore a cancelled booking or keep it buried — was settled by the package's own sources (AC-3's record history "created → moved → cancelled → restored"; §F1's "returns the record to active"; the `(source, externalRef)` uniqueness rule), all pointing the same way.

- **P1 (fixes F1) — suspected-partial operationalised in §F3:** the complete-day median is the median of daily calorie totals over complete days in the last 28 days, requiring at least 3 complete days (otherwise the calorie signal is unavailable, never guessed). A partial day flags when at least one meal is logged and at least one is missing **and** its total is below 50% of that median, **or** the only logged meal is breakfast. Evaluated on every macro-log write and recomputed at Ask Queue composition; the writer is the macro-log write path / queue read; the macro refresh never writes the flag; flagging never writes completeness. New surface: AC-60 / N42.
- **P2 (fixes F2) — restore semantics in §F1:** a payload for a tombstoned record restores it when it is a genuine new booking event — `cancelled: false`, a date/time different from the tombstoned record's last state, and (where the source supplies a timestamp) a timestamp **later than the cancellation event**; an identical or earlier-or-equal payload is stale → `tombstoned-kept`, no change. A restore keeps the same `uid`, clears the tombstone, increments revision, and appends a `cancelled → restored` history row. Two surfaces added to the action catalogue: the sync-side restore inside `upsertSyncedSession`, and a user-confirmed `restoreFromSnapshot(snapshotId)` restoring by idempotent replay/read-back. New surfaces: AC-61 / N43, AC-62 / N44.
- **P3 — training-day completeness exposure:** the Train read returns per-day training completeness `{date, state, basis}`; the field's home is a per-day aggregate, not a per-session column. AC-63 / N45.
- **P4 — job record surface:** the automation-consent/job record gains `owner` and `lastSuccessAt`, exposed via `getFrameworkStatus`. D7's runtime legs were ruled document-verified by design (the replica installs no live jobs; the schedule, failure and staleness behaviour is fully specified in `automation.md`); its surface legs became testable. AC-64 / N46.
- **P5 — reconciliation surface:** a read-only `getReconciliationReport(weekStart)` action returning the weekly report structure (duplicates, missing days, mismatched totals, failed syncs, not-complete days naming the missing meals). AC-65 / N47.
- **P6 — clarifications bundle (AC-66 / N48):** macro-log reads surface `provStatus`, `completenessProvStatus`, `source`, `sourceRef`, `revision`; the oldest-first rule for multiple pending confirmations stated (one per night); `getTodayDashboard` summary points are computed as-of the requested date; muscle states are maintained at write time (stated as known behaviour); the estimates-permission enforcement split (chat flow, not app layer) cross-referenced honestly; backup-manifest entries must show a tombstoned record's true state; the goal-maths rounding order specified (protein and fat rounded to whole grams first; carbs computed from the rounded values); and `provStatus` transitions are themselves `record_history` rows (build fix).

The gate also ruled that the test-worn mock install **had to be restored to its canonical state before any final packaging evidence** — Phase 3 probes had permanently moved it (an undeletable probe weigh-in had shifted the live recommendation; counts were inflated; a target's provenance had flipped as a disclosed tester side effect). A canonical re-seed (R1) was prescribed, with the unreachable test states closed procedurally *during* seeding (see §8).

## 7. Build fixes, re-test, and the final scoped re-test

**Build fixes (B1–B5)** implemented P1–P6 in the replica: live suspected-partial evaluation; the restore trigger and `restoreFromSnapshot`; per-day training completeness in the Train read; `owner`/`lastSuccessAt` on the job record; `getReconciliationReport`; and the P6 read-contract bundle. **Re-seed R1** then rebuilt the canonical mock state in the fixed build's final form.

**Gate G3 recheck** (independent, own fixtures) passed everything in its scope except one precisely-scoped defect: D6/N42, D8/N44, N39, D7 surface legs/N46, N14/N47, N20/N45 and N48 all **PASS**; the staged closures were verified (§8); the canonical manifest matched exactly; regression spots T1, T5, T10, N19, N22 all **PASS**. The single failure: the first restore build enforced `cancelled: false` + a changed date/time but **not** the source-timestamp condition — a stale payload with a changed date resurrected the record whether its timestamp was earlier than the cancellation's or equal to it (T3 / N43).

**Fix:** the tombstoned branch now retains the cancellation event's source timestamp at tombstone time (overwritten at each new tombstoning, cleared on restore) and requires the incoming payload's timestamp to be **strictly later** for a restore; earlier-or-equal returns `tombstoned-kept` with no change, no revision increment and no history row.

**Final re-test (T3 / N43, independent, fresh fixture): PASS on every leg** — a genuine re-booking restored the same record (same `uid`, `cancelled → restored` history row); an earlier-timestamp stale payload and an equal-timestamp stale payload, each with a changed date, both returned `tombstoned-kept` with zero changes and zero new history rows; a later-timestamp payload still marked cancelled never restored; a later-timestamp payload with an *unchanged* date/time was kept, exactly as the conjunction rule requires; and no duplicate record existed at any point. Probes were measured against the *second* cancellation's timestamp, independently confirming the retained timestamp is overwritten at each tombstoning.

**Gate G3 verdict: PASS in full.**

## 8. Canonical mock manifest and staged-closure evidence

The canonical mock state (fictional user Alex Tan) against which the preservation diff and all final evidence are taken:

| Measure | Canonical value |
|---|---|
| Sessions | 9 active / 1 tombstoned |
| Macro days | 27 |
| Body-metric days | 25 |
| Performed sets | 72 |
| Exercise baselines | 43 |
| Plans | 3 |
| Record-history rows | 9 |
| Sync-log rows | 8 |
| Backup snapshots | 0 |
| Notes | 2 |
| Macro target | 2510 kcal / protein 155 g / carbs 315 g / fat 70 g, `provStatus: "derived"` |
| Live recommendation maintenance | 2,643 kcal (latest weight 75.5 kg × step factor 35; step average 9,255 over 12 logged days) |

This manifest was verified independently at the start of Phase 3 (T10), after the R1 re-seed, after the G3 recheck's probes and cleanup, and after the final T3/N43 re-test's cleanup (`manifestMatch: true` on the prescribed reseed helper each time, then re-verified from fresh reads). The seeded "the press" pending confirmation remains pending and unlogged throughout — the owner's confirmation rule, holding in the fixture itself.

**Staged closures during the R1 re-seed** (states unreachable on a seeded install, closed in seeding order and independently ruled genuinely evidenced at the G3 recheck):

- **N1 onboarding replay** — a scripted from-empty onboarding run for the second fictional persona ("Riley Mock"): the exact block order (charter → automation → connectors → basics/training → wants → goals → remaining blocks → capability tour → read-back), the exact save question verbatim, an edit with re-confirmation of only the changed part, a skipped question stored blank, and saving only on the exact yes. Independent residue check afterwards: no trace of the replay persona in any read surface; the install reads as the canonical Alex Tan state only.
- **N4 / N6 / N7 unavailable branches** — no-goal-mode, no-weight ("log your weight first") and fewer-than-3-logged-step-days states each captured with their reason naming the missing input; every reachable branch had already been verified live in Phase 3, and the canonical recommendation figures were recomputed independently from the raw step rows (sum 111,059 / 12 = 9,255; 75.5 × 35 = 2,643).
- **Step-factor bands** — band 35 verified live on canonical data; bands 31, 33 and 37 constructed from scratch rows during seeding and recomputed exactly from the log's own inputs (77.2 × 31 = 2,393; × 33 = 2,548; × 37 = 2,856).
- **EWMA fewer-than-2-weigh-ins edge** — with exactly one weigh-in, the trend reads unavailable; the full-history trend was independently re-verified at all 8 canonical points.
- **D6 empty-queue leg** — an empty queue (0 items) captured on a complete, nothing-outstanding day *before* the standing pending confirmation was seeded; independently corroborated afterwards (that date's queue now returns exactly the one later-seeded item).

Three builder caveats from the fix round were ruled acceptable by the recheck: clearing a legacy hand-seeded flag on a complete day (the P1 rule can never flag a complete day, so retaining it would have violated the package); the empty-queue construction above (the only correct one — a no-data day always generates the missing-food slot); and not-logged days stored as `completeness: "none"` rows with null values (every read surface treats them as gaps; values are null, never 0).

Two tester observations were recorded without failing anything: a flagged day's suspected-partial queue item also appears in the next day's queue, correctly date-labelled (consistent with the framework's ask-and-wait, ask-again-next-day role); and the weight *chart* series on a pre-data dashboard still returns full history while its summary points are correctly as-of (the package's as-of rule is written for summary points; chart date-clipping is a candidate future clarification).

## 9. What remains outstanding

1. **Gate G2 item 6 — the exercise-library screenshot.** One phone capture of the replica's Train tab scrolled to the exercise library grid. Evidence only; no code change is indicated. Until it exists, Gate G2 is not closed in full and this package's verification is recorded as: **all functional, framework, NUX and Phase 3/G3 testing PASS; visual evidence complete except the library grid.**
2. Nothing else. The Phase 4 packaging steps (this report's inclusion, a final privacy sweep of the shipped package, and the distribution bundle) are packaging, not further verification.

*End of test report.*
