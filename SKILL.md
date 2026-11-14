---
name: fitness_tracker_2_replica
description: All-in-one fitness tracker for Muse — workout logging, exercise baselines, a body heat map that updates automatically from logged sessions, recommended workouts from ready muscles, goal-linked macros, steps, weight trend and optional daily automation. Built as a Muse Space. Onboards each new user from zero, including their goals, never invents a weight, target or machine, confirms ambiguous machines with photos, syncs without duplicating, and keeps provenance and day completeness on every record.
---

# BestFit — All-in-One Fitness Tracker Skill (v3)

One tracker for training AND nutrition. No separate apps. Built as a Muse Space with five sections, in this order: **Today / Train / Calendar / Macros / More**.

**Top priority, above speed and helpfulness: LOGGING ACCURACY.** A record is only ever what the user actually gave or confirmed. A blank is a correct answer. A guessed number is a wrong answer, even if it happens to be close.

Companion files in this package: `PRD.md` (product requirements and acceptance criteria), `design-specs.md` (screens, charts, layout), `framework.md` (data layer F1–F5, full data model), `automation.md` (job templates), `onboarding.md` (the full onboarding script, run verbatim), `onboarding-prompt.md` (the packaged onboarding prompt the user personalises themselves first), `catalog.md` (exercise catalogue, muscle map, look-alike pairs), `guardrails.md` (the rule card), `DECISIONS.md` (design decisions made for this generic product).

**This is a framework, not a fixed app.** The product's name is **BestFit** — fitness covering training, macros/food and steps in one framework each user shapes around their own life. Everything is customisable — theme/colours, targets, goal mode, exercise catalogue, meal structure, every automation job and its time — at onboarding and at any time after, by telling the agent. Every default is a starting point; nothing is locked, and a change never rewrites logged history. New users are told this first, in the onboarding charter, before any setup question.

## What it tracks

1. **Workouts**
   - Sessions: date, title/type, status (scheduled / completed / skipped), exercises per session.
   - Per exercise: name, sets, reps, weight, note. Any field the user does not know stays **blank**.
   - Set-level records underneath each session (see `framework.md`, F4) so progression becomes computable.
   - Baselines: the user's current working weight per exercise, grouped Push / Pull / Biceps / Lower Body / Core / Misc / Cardio.
   - Plans: reusable workout plans (for example Upper, Lower, Push, Pull, Legs, Full Body) built from the user's own baselines.
   - Calendar: planned sessions and completed sessions in one view.

2. **Body heat map — two layers, always stated as two layers**
   - 18 muscle groups, professional names only: Pectorals, Front Deltoids, Side Deltoids, Rear Deltoids, Trapezius, Upper Back, Lats, Lower Back, Biceps, Triceps, Forearms, Abs, Obliques, Glutes, Quads, Hamstrings, Adductors, Calves.
   - Front and back body figures, soft gradient design (not square blocks): red = Sore, amber = Recovering, green = Ready.
   - **Data layer:** every muscle state carries `hasHistory` and `lastWorked`. A muscle that has never been worked in a logged session has `hasHistory = false` and is **"no data yet"** in the data. Never-logged is not the same as recovered.
   - **Display layer:** the screen renders a never-logged muscle as Ready. That display is frozen as-is; surfacing the no-data distinction on screen is a separate future design decision, not part of this replica.
   - **Automatic:** derived from logged, completed sessions only. The user does not maintain it by hand.
   - Manual tap is an override only: it wins while it is newer than the last session that worked that muscle, and lapses after 3 days.

3. **Recommended workout**
   - Built only from muscles currently Ready. Nothing sore or recovering gets loaded.
   - Max 6 exercises, priority: Pectorals, Lats, Upper Back, Quads, Glutes, Hamstrings, Front Deltoids, Side Deltoids, Triceps, Biceps, Lower Back, Rear Deltoids, Trapezius, Adductors, Calves, Forearms, Abs, Obliques.
   - Each exercise names its machine/setup and shows a photo (see Machine Photo Rule).
   - Starting weight = the user's own baseline if one exists. If no baseline exists: "No baseline yet — leave blank / build up by feel". Never a guessed number.
   - A recommendation is never auto-logged.

4. **Macros — all in one, same tracker**
   - One daily target: calories, protein (g), carbs (g), fat (g), water (ml), notes. Every field starts **blank/null** until the user sets it or accepts a calculated target in onboarding.
   - Daily macro logs: date + calories, protein, carbs, fat, water, notes. Missing values stay blank, never 0.
   - Plans and logs sit next to training, so a training day and its food are visible together.
   - Never invent a calorie or protein target. Never backfill a day the user did not report. Never overwrite a user-supplied figure with an estimate.

5. **Body dashboard (inside Macros)**
   - Steps per day, from the steps source the user chose in onboarding, or recorded manually, or blank.
   - Weight per day, shown against the user's own target weight if they set one.
   - A Recommendations block: goal-mode based recommended macros (see Goals below), display-only until the user applies them with Apply + Confirm. If no weight is logged, the block shows an honest "log your weight first" state instead of a computed recommendation.

6. **Goals**
   - The user's goal in their own words, stored word for word.
   - What the user wants from the tracker itself, in their own words (onboarding Stage C), with an honest out-loud mapping of which wants the tracker can and cannot meet.
   - An optional goal mode: Lose Fat, Recomposition, Gain Muscle, or Gain Weight (multipliers on maintenance are defined in `framework.md` and `DECISIONS.md`).
   - Optional target weight, pace preference, daily step goal, and a 12-week success description, all in the user's own words or numbers, all blank if not given.
   - The selected goal mode drives the Today steer and the morning brief recommendation. Changing the mode changes defaults from that day on; it never rewrites logged history.

7. **Today dashboard**
   - A daily dashboard: 3-point executive summary, summary table, macro / steps / weight charts, a week-on-week table, and ONE priority recommendation. Full component order and rules are in `design-specs.md`.

## How it tracks (the user experience)

- The user just tells Muse in chat or by voice what they did: "Today I did lat pulldown, rows and curls".
- Muse asks only for what is missing and material (see Onboarding + Session Logging Flow), then logs it.
- The heat map, recommendations and baselines update from that log. No forms are the primary interface.

## Onboarding — ask, do not assume; make it personal

Run `onboarding.md` in full for every new user. It is the package onboarding script, used verbatim. **Order matters — and the order is the experience order, automation first:**

0. **Opening + the framework charter** — the user hears first that BestFit is a framework and everything in it can be customised, now or at any time later.
1. **Stage A — Automation first (Block 8)** — each automation explained (what it does, when it runs, what it sends) and separately consented, off until the user says yes to that one, at a time the user chooses. The timezone is recorded here, because every time in this stage is in it.
2. **Stage B — Connectors** — what to connect, each named with what it unlocks in one line: their food source (macro refresh), their health platform (steps sync), their calendar (booking mirror → scheduled sessions), their email inbox (booking confirmations path). Each optional, none assumed; choices recorded per the `framework.md` §F2 source keys.
3. **Basics (Block 1)** and **Training life (Block 2)** — name, unit, and how training fits their week and their life (including the days that fit, in their words).
4. **Stage C — "What do you want from this tracker?"** — their answer verbatim (`trackerWants`), mapped honestly to what the tracker can and cannot do; out of scope is said out loud, never promised. Then **Goals (Block 3)** — own words, goal mode, target weight, pace, step goal, 12-week success — and **Goal-linked macros (Block 4)** — user-given targets, or a calculated target shown in full and saved only on a yes, or blank.
5. **Equipment (Block 5)**, **Baselines (Block 6)** — exercise by exercise, with the three-answer rule and the Photo Rule — and **Food reporting style and estimates permission (Block 7)**, including their meal structure.
6. **Stage D — Capability tour** — what the tracker is capable of, in plain language, closing with the charter restated.
7. **Read-back and save (Block 9)** — their goal and wants read back in their exact words; save only on an unambiguous yes to the exact question: **"Ready for me to save this? Reply yes to save, or tell me what to change."** A "yes" answering any other question is never consent to save.

**Personal, not a form:** use the user's name naturally once known; play their own words back to them; shape questions around their meals, their routine, their week; one topic at a time, at their pace, with pausing allowed; anything skipped stays blank. The package ships `onboarding-prompt.md` — a copy-paste prompt the user personalises **themselves** first (their meals, their goals, their choices). A user who arrives with it filled in is confirmed part by part, not re-asked from zero; its blanks are asked, never assumed. The personal context this creates is also what the morning brief, the Ask Queue and the Today steer speak to: the user's stated goals and wants, in their words, where the framework's data supports it — never invented motivation, never a generic template wearing their name.

## Session logging flow (every time)

1. Confirm the session actually happened. "I will do legs tomorrow" = scheduled, not completed. Only completed sessions drive the heat map.
2. For each exercise, capture: exact name, sets, reps, weight + unit, note.
3. Missing weight / sets / reps → ask once. Still unknown → **blank**. Never 0, never "bodyweight" unless it truly was bodyweight, never last session's weight copied forward without saying so and getting a yes.
4. Ambiguous exercise or machine → Machine Photo Rule. Do not log until resolved, or log the exercise name exactly as the user described it with note "machine unconfirmed" and weight blank.
5. Every save follows the confirm-before-save transaction (F5, `framework.md`): parse → proposed record → one material clarification if needed → confirmation bound to that exact revision → atomic save → read-back receipt in chat. Any edit voids the earlier confirmation.
6. After logging, state what changed: which muscles the heat map will now mark sore/recovering, and what looks good to hit next.

## Sync — the F1 contract (not the old de-dupe wording)

Full detail in `framework.md`. In short:

- Every session and macro log carries a stable, never-reused `uid`, a `revision`, `updatedAt`, and a tombstone `deletedAt`. There is a unique constraint on `(source, externalRef)`.
- Sync matches in this order: **externalRef → uid → fuzzy key**. A fuzzy match with a changed date (a reschedule) updates the one record and logs the move. An unresolved fuzzy match goes to an **ambiguity queue** — never auto-merged.
- Every mutation carries an idempotency key; replaying a sync is a no-op.
- **Field ownership:** sync may update source-owned fields (date, time, title, status = scheduled/cancelled) and may **never** overwrite user-owned fields (logged exercises, weights, user corrections, completed status).
- Anything arriving from calendar or email is **scheduled only**. Only the user's confirmation of attendance makes it completed.
- Deletes are tombstones, not hard deletes, so a stale source cannot resurrect a cancelled session.
- Every record carries provenance (canonical field `provStatus`: observed / estimated / derived) and every day carries a completeness state (none / partial / complete). A food day is complete only when breakfast, lunch and dinner are all logged, or the user confirms the day — complete by the fact of the logged meals; the food-source refresh never writes completeness, and the system never closes or locks a day on its own, so a day with missing meals stays open and is asked about gently ("What did you have for lunch?"). Training-day completeness is user-asserted only. Averages run over complete days only, with the coverage denominator always printed.

Do NOT use the superseded pre-upgrade de-dupe description (matching on date + session/exercise name alone). That keying was the reschedule-duplication defect F1 replaces.

## Exercise → muscle mapping

Keyword based, in this exact order (order prevents the classic errors):

- Leg Extension → Quads (primary)
- Leg Curl → Hamstrings (primary). Check this BEFORE generic "curl", or leg curl becomes Biceps
- Wrist Curl → Forearms. Check this BEFORE generic "curl", or wrist curl becomes Biceps
- Curl (other) → Biceps; Hammer Curl also → Forearms
- Rear Delt Fly / Reverse Fly / Reverse Pec Deck → Rear Deltoids. Check this BEFORE generic "fly / pec deck", or reverse pec deck becomes Pectorals
- Chest / Bench / Fly / Pec Deck → Pectorals. NOTE: "press" or "bench" alone is ambiguous for machine ID (see Photo Rule), even though the muscle is Pectorals
- Lat Pulldown / Pull-up / Chin-up → Lats. Pulldown does NOT mark Biceps sore — the user may still train arms after a back day
- Row → Lats + Upper Back (primary); Lower Back + Trapezius (secondary)
- Deadlift → Lower Back + Glutes + Hamstrings + Trapezius
- RDL / Romanian → Hamstrings + Glutes + Lower Back
- Back Extension → Lower Back
- Shrug → Trapezius
- Tricep / Pushdown / Skull Crusher / Dip → Triceps
- Shoulder Press / Overhead / Military Press → Front Deltoids + Side Deltoids
- Lateral Raise → Side Deltoids; Front Raise → Front Deltoids
- Face Pull → Rear Deltoids + Trapezius
- Squat / Leg Press / Lunge / Step-up → Quads + Glutes (primary); Hamstrings + Adductors (secondary)
- Hip Adductor → Adductors. Hip Abductor → Glutes. These are opposites — never swap them
- Hip Thrust → Glutes
- Sled Push → Quads + Glutes + Calves
- Bike / Cycling / Assault Bike → Quads + Glutes + Calves
- Calf Raise → Calves
- Sit-up / Crunch / Leg Raise / Knee Raise / Captain's Chair / Plank → Abs
- Twist / Oblique / Woodchop → Obliques + Abs (secondary)

Unmapped exercise name → ask the user which muscle it worked / show photos. Do not force-map it by guessing.

## Heat map cooldown

- Primary muscle: worked 0–1 days ago = Sore, 2 days ago = Recovering, 3+ days = Ready
- Secondary muscle: 0–1 days ago = Recovering, 2+ days = Ready (one step milder; a secondary muscle is never Sore from this rule)
- Never worked in logged sessions: data layer = "no data yet" (`hasHistory = false`); display layer = Ready (see the two-layer rule in "What it tracks")
- Manual override: wins only if newer than the last session that worked it; lapses after 3 days

## Machine Photo Rule — do not speculate (hard guardrail)

Whenever Muse is not 100% sure which machine or movement the user means, STOP and show photos first. Ask: "Which one is it?" Log nothing specific until the user picks.

Always trigger on these look-alike pairs (show ALL options, labelled "✓ This one" / "✗ Not this one"):

- Smith machine bench press vs free barbell bench press
- Smith machine squat vs barbell back squat
- Smith machine shoulder press vs machine (stack) shoulder press vs dumbbell shoulder press
- Hack squat vs leg press
- Chest-supported T-Bar row vs seated cable row vs Smith row vs machine row
- Hip adductor (pads push in) vs hip abductor (pads push out)
- Leg extension vs leg curl
- Close-grip Smith bench (triceps) vs normal Smith bench (pectorals)

Also trigger on vague language: "the press", "a row machine", "the leg machine", "chest thing", "some Smith thing", "low row, maybe on the Smith". Vague = photos, not a guess.

Photo sourcing: the shipped source is **`photos/` in this package** — the verified generic photo set with its index (`photos/photos.md`), copied into the Space at setup so the photos always load. Use a real photo of the actual machine/movement, visually verified. Never use an illustration or render as a machine ID photo. If no confident photo exists for a movement, show no photo rather than a wrong one, and ask the user to describe or photograph their machine.

## Hard guardrails (never break)

1. **Blank means blank.** Unknown weight, reps, target or bar weight stays empty. Never 0, never a typical value, never another user's value.
2. **No invented numbers.** No baseline, macro target, calorie estimate-as-fact, machine bar weight, or progression is created without the user's own number or explicit yes to a labelled suggestion or a shown calculation.
3. **No machine speculation.** Ambiguous machine = photos + question first (above).
4. **No completed-by-assumption.** Mentioned / planned / scheduled ≠ completed. Confirm attendance before logging as done.
5. **User's number wins.** If the user corrects any figure, replace it with theirs and scale nothing else silently. The prior value stays visible in the record history.
6. **No personal data in this package.** This bundle contains zero names, weights, schedules, trainers, targets or health data from any existing user. Every install starts empty.
7. **Safety.** Record injuries/pain as stated. Do not prescribe maximal loads, rehabilitation, or medical/nutrition diagnoses. Suggest seeing a qualified professional for pain or a medical goal.
8. **Privacy.** A user's tracker data is theirs alone. Never copy one user's baselines, macros or sessions into another user's tracker.
9. **Estimates only with permission.** A food estimate requires the user's recorded estimates permission, is always labelled estimated (`provStatus = estimated`), and never overwrites a user-supplied figure silently.
10. **Complete days only for averages.** Averages and hit rates are computed over complete days only and always printed with their coverage denominator. Partial days are visible, labelled partial, and excluded.

## Template defaults — zero means "not set", in three contexts

A fresh template install seeds the starter exercise catalogue with every baseline weight at **0**.

- In the template seed, 0 is a placeholder for "not set yet". It is NOT a weight the user lifted, NOT a starting recommendation, and never a logged weight.
- Everywhere logic runs (recommendations, progression, read-backs) a 0 baseline is treated exactly like blank: "No baseline yet".
- The moment a user gives a real weight, that number replaces the 0. If they never give one, it stays 0 / not set.
- Session logging is different from template defaults: a weight in a logged session is only a number the user actually gave in that session. Unknown in a session = blank, never 0.

## Self-improving — the tracker learns what the user adds

The tracker improves from the user's own additions, automatically:

- **New exercise added** → it joins their catalogue and baselines (weight = given, or 0 / not set), appears in plans, logging and recommendations, and is remembered under the user's own name for it.
- **User's own name for a known exercise** ("the leg push thing" = their Leg Press) → remember that alias for that user only; next time, recognise it without re-asking, but still show the photo once if the alias was first used ambiguously.
- **New vague phrase resolved by photo pick** → that phrase joins that user's photo-check triggers, so the same words get the same careful check next time, not a silent guess.
- **Repeated choice** (they always pick one setup option, they always train a muscle a certain way) → surface it once as a noticed pattern and, if they confirm, apply it as their default going forward.
- Nothing learned from one user ever carries into another user's tracker.

## Help me choose — when the user is not sure what to add or do

If the user is unsure which exercise to add, or which to do:

1. Ask one short question: what do you want to train, or what equipment is in front of you / available?
2. **Briefly describe** 2–4 candidate exercises: one line each on what it works and the machine or setup it uses. Keep it short, no lectures.
3. **Recommend one** based on the user's own description (muscle wanted, equipment available, what is Ready on their heat map), and say why in one line. Label it a recommendation.
4. Give the others as examples they can pick instead.
5. They pick → then log or add it. A recommendation is never auto-logged, and if their pick is still machine-ambiguous, the Photo Rule applies before logging.

## Calendar sync — one calendar, every logging channel

The tracker calendar is the single synced view, fed by every channel the user logs through, under the F1 contract:

- **Verbal / voice / chat:** a session the user reports is logged in the tracker and appears on the tracker calendar for that date.
- **Calendar:** (only if the user consented to calendar booking sync in onboarding) training bookings on the user's calendar appear in the tracker calendar as scheduled sessions.
- **Email:** (only if the user consented to email booking sync in onboarding) booking confirmations, reschedules and cancellations received by email update the tracker calendar to match (booked = scheduled, moved = moved, cancelled = tombstoned/cancelled).
- **De-duplication:** channels feed one record, matched in F1 order (externalRef → uid → fuzzy key). An unresolved possible duplicate goes to the ambiguity queue and the user is asked; a second copy is never silently created.
- **Accuracy gate:** anything arriving from calendar or email is **scheduled only**. Only the user's confirmation that they attended makes it completed, and only completed sessions drive the heat map and recommendations.
- The end-to-end booking path specified for this product is defined in `DECISIONS.md` (O7).

## Extensibility — the list is a starting set, never a cap

The exercise catalogue and a user's baseline list are starting sets only. There is no limit on exercises or baselines.

- When a user logs an exercise that is not in their baselines yet, add it as a new baseline entry: weight = what they gave in that session, or **blank** if they gave none.
- When a user names a new exercise during onboarding or later, add it — never refuse it because it is "not in the list".
- The catalogue grows the same way: a confirmed new exercise (with its muscle mapping confirmed by the user, and its machine photo if one is needed) joins the catalogue for that user.

## Data model (Space)

Summary — the full model, with framework fields, is in `framework.md`.

Tables: `baselines`, `workout_plans`, `sessions`, `macro_target`, `macro_logs`, `muscle_states`, `notes`, plus the framework tables `performed_sets`, `body_metrics`, `record_history`, `sync_log`, `ambiguity_queue`, and the goals/consent tables designed in `framework.md` (`goals`, `goal_recommendation_state`, `automation_consents`, `user_profile`).

Exercise item: `{ name, sets, reps, weight, note }` — sets/reps/weight are strings so "blank", units and ranges like "8-12" survive intact.
Dates are the user's local calendar days (YYYY-MM-DD) in their timezone from onboarding.

## Deliverable in this package

- `SKILL.md` — this operating skill
- `PRD.md` — product requirements, acceptance criteria
- `design-specs.md` — screens, components, charts
- `framework.md` — the F1–F5 framework and full data model
- `automation.md` — job templates (all times and channels are onboarding parameters)
- `onboarding.md` — the full question script, word for word, including goals, in the experience order (automation first, connectors, wants, capability tour)
- `onboarding-prompt.md` — the packaged onboarding prompt the user personalises themselves (meals, goals, choices), shipped with the skill
- `catalog.md` — exercise catalogue, muscle map, look-alike photo pairs, recommendation catalogue
- `guardrails.md` — the one-page rule card
- `DECISIONS.md` — the decision log for this generic product
- `photos/` — the verified generic photo set (18 photos) with its index `photos/photos.md`: the shipped source for every machine/exercise photo; copied into the Space at setup
