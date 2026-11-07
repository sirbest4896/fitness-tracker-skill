---
name: fitness_tracker
description: All-in-one fitness tracker for Muse — workout logging, exercise baselines, a body heat map that updates automatically from logged sessions, recommended workouts from non-sore muscles, and macro tracking. Built as a Space. Onboards each new user from zero, never invents a weight, target or machine, and confirms ambiguous machines with photos.
---

# Fitness Tracker — All-in-One Skill (Generic SKU v2.1)

One tracker for training AND nutrition. No separate apps.

**Top priority, above speed and helpfulness: LOGGING ACCURACY.** A record is only ever what the user actually gave or confirmed. See the Hard Guardrails.

## What it tracks

1. **Workouts**
   - Sessions: date, title/type, status (scheduled / completed / skipped), exercises per session
   - Per exercise: name, sets, reps, weight, note. Any field the user does not know stays **blank**
   - Baselines: the user's current working weight per exercise, grouped Push / Pull / Biceps / Lower Body / Core / Misc / Cardio
   - Plans: reusable workout plans (e.g. Upper, Lower, Push, Pull, Legs, Full Body) built from the user's own baselines
   - Calendar: planned sessions and completed sessions in one view

2. **Body heat map**
   - 18 muscle groups, professional names only: Pectorals, Front Deltoids, Side Deltoids, Rear Deltoids, Trapezius, Upper Back, Lats, Lower Back, Biceps, Triceps, Forearms, Abs, Obliques, Glutes, Quads, Hamstrings, Adductors, Calves
   - Front and back body figures, soft gradient design (not square blocks): red = Sore, amber = Recovering, green = Ready
   - **Automatic**: derived from logged, completed sessions only (see Mapping + Cooldown below). The user does not maintain it by hand
   - Manual tap is an override only: it wins while it is newer than the last session that worked that muscle, and lapses after 3 days

3. **Recommended workout**
   - Built only from muscles currently Ready. Nothing sore or recovering gets loaded
   - Max 6 exercises, priority: Pectorals, Lats, Upper Back, Quads, Glutes, Hamstrings, Front Deltoids, Side Deltoids, Triceps, Biceps, Lower Back, Rear Deltoids, Trapezius, Adductors, Calves, Forearms, Abs, Obliques
   - Each exercise names its machine/setup and shows a photo (see Machine Photo Rule)
   - Starting weight = the user's own baseline if one exists. If no baseline exists: "No baseline yet — leave blank / build up by feel". Never a guessed number

4. **Macros — all in one, same tracker**
   - One daily target: calories, protein (g), carbs (g), fat (g), water (ml), notes. Every field starts **blank/null** until the user sets it
   - Daily macro logs: date + calories, protein, carbs, fat, water, notes. Missing values stay blank, never 0
   - Plans and logs sit next to training, so a training day and its food are visible together
   - Never invent a calorie or protein target. Never backfill a day the user did not report. Never overwrite a user-supplied figure with an estimate

## How it tracks (the user experience)

- The user just tells Muse in chat or by voice what they did: "Today I did lat pulldown, rows and curls"
- Muse asks only for what is missing and material (see Onboarding + Session Logging Flow), then logs it
- The heat map, recommendations and baselines update from that log. No forms are the primary interface

## Onboarding — ask, do not assume

Run `onboarding.md` in full for every new user. Order matters. Short version:

**Block 1 — Basics**
1. What should the tracker call you? (display name only, optional)
2. Timezone? Unit preference: kg or lb? (never convert silently; store as given, label the unit)

**Block 2 — Training**
3. Do you train now? How many times a week, roughly? What kind: gym machines, free weights, classes, PT, home, cardio?
4. What split or routine do you follow, if any? (Push/Pull/Legs, Upper/Lower, Full Body, PT-led, none yet)
5. What is your goal in your own words? (strength, muscle, fat loss, general fitness, other)
6. Any injury, pain or movement you avoid? If they say none, record "none stated" — do not clear them for anything, just record it
7. Preferred sets/reps scheme? If they do not have one, offer the common starting template "3 sets of 8-12" as a *suggestion they can accept or change* — never present it as their scheme until they accept it

**Block 3 — Equipment**
8. Does your gym have a Smith machine? If yes: do you know its bar starting weight? If not known: **leave blank**. Do NOT use any other user's bar weight
9. Which machines do you usually have access to? (free text, optional)

**Block 4 — Baselines, exercise by exercise**
10. Go group by group (Push, Pull, Biceps, Lower, Core, Cardio). For each common exercise ask: "Do you do this one? What weight, sets and reps last time?"
    - Knows it → record exactly as given, with unit
    - Does it but does not know the weight → record the exercise, weight = **blank**
    - Does not do it → skip it, do not add it to their baselines
    - Unsure which machine it is → Machine Photo Rule, before recording anything

**Block 5 — Macros**
11. Do you want to track macros in the same tracker? If no, leave the whole Macros section blank and skip it in daily use
12. If yes: do you have targets already (calories / protein / carbs / fat / water)? Record only what they give. Anything they do not give stays **blank** — never calculate, suggest-as-fact, or fill a "typical" target without labelling it a suggestion and getting a yes
13. How will they report food: exact numbers, app totals they read out, or meal descriptions? If meal descriptions, Muse may estimate ONLY if the user agrees to estimates, and every estimate must be labelled "estimate" and never overwritten silently later

**Block 6 — Confirm before first save**
14. Read back a summary: training days, split, goal, unit, Smith bar weight (or "blank — not known"), how many baselines recorded vs left blank, macro tracking on/off and which target fields are set vs blank
15. Save only after they confirm. A planned, mentioned or intended workout is NOT a completed session

## Session logging flow (every time)

1. Confirm the session actually happened. "I will do legs tomorrow" = scheduled, not completed. Only completed sessions drive the heat map
2. For each exercise, capture: exact name, sets, reps, weight + unit, note
3. Missing weight / sets / reps → ask once. Still unknown → **blank**. Never 0, never "bodyweight" unless it truly was bodyweight, never last session's weight copied forward without saying so and getting a yes
4. Ambiguous exercise or machine → Machine Photo Rule. Do not log until resolved, or log the exercise name exactly as the user described it with note "machine unconfirmed" and weight blank
5. After logging, state what changed: which muscles the heat map will now mark sore/recovering, and what looks good to hit next

## Exercise → muscle mapping

Keyword based, in this exact order (order prevents the classic errors):

- Leg Extension → Quads (primary)
- Leg Curl → Hamstrings (primary). Check this BEFORE generic "curl", or leg curl becomes Biceps
- Curl (other) → Biceps; Hammer Curl also → Forearms
- Wrist Curl → Forearms
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
- Rear Delt Fly / Reverse Fly / Reverse Pec Deck → Rear Deltoids
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

- Primary muscle: worked 0-1 days ago = Sore, 2 days ago = Recovering, 3+ days = Ready
- Secondary muscle: 0-1 days ago = Recovering, 2+ days = Ready (one step milder)
- Never worked in logged sessions = Ready
- Manual override: wins only if newer than the last session that worked it; lapses after 3 days

## Machine Photo Rule — do not speculate (hard guardrail)

Whenever Muse is not 100% sure which machine or movement the user means, STOP and show photos first. Ask: "Which one is it?" Log nothing specific until the user picks.

Always trigger on these look-alike pairs (show BOTH, labelled "✓ This one" / "✗ Not this one"):
- Smith machine bench press vs free barbell bench press
- Smith machine squat vs barbell back squat
- Smith machine shoulder press vs machine (stack) shoulder press vs dumbbell shoulder press
- Hack squat vs leg press
- Chest-supported T-Bar row vs seated cable row vs Smith row vs machine row
- Hip adductor (pads push in) vs hip abductor (pads push out)
- Leg extension vs leg curl
- Close-grip Smith bench (triceps) vs normal Smith bench (pectorals)

Also trigger on vague language: "the press", "a row machine", "the leg machine", "chest thing", "some Smith thing", "low row, maybe on the Smith". Vague = photos, not a guess.

If no confident photo exists for a movement, show no photo rather than a wrong one, and ask the user to describe or photograph their machine.

## Hard guardrails (never break)

1. **Blank means blank.** Unknown weight, reps, target or bar weight stays empty. Never 0, never a typical value, never another user's value
2. **No invented numbers.** No baseline, macro target, calorie estimate-as-fact, Smith bar weight, or progression is created without the user's own number or explicit yes to a labelled suggestion
3. **No machine speculation.** Ambiguous machine = photos + question first (above)
4. **No completed-by-assumption.** Mentioned / planned / scheduled ≠ completed. Confirm attendance before logging as done
5. **User's number wins.** If the user corrects any figure, replace it with theirs and scale nothing else silently
6. **No personal data in the SKU.** The shipped bundle contains zero names, weights, schedules, trainers, targets or health data from any existing user. Every install starts empty
7. **Safety.** Record injuries/pain as stated. Do not prescribe maximal loads, rehabilitation, or medical/nutrition diagnoses. Suggest seeing a qualified professional for pain or a medical goal
8. **Privacy.** A user's tracker data is theirs alone. Never copy one user's baselines, macros or sessions into another user's tracker

## Template defaults — all zero, and zero means "not set"

A fresh template install seeds the starter exercise catalog with every baseline weight at **0**.
- 0 is a placeholder for "not set yet". It is NOT a weight the user lifted, NOT a starting recommendation, and never a logged weight
- Everywhere logic runs (recommendations, progression, read-backs) a 0 baseline is treated exactly like blank: "No baseline yet"
- The moment a user gives a real weight, that number replaces the 0. If they never give one, it stays 0 / not set
- Session logging is different from template defaults: a weight in a logged session is only a number the user actually gave in that session. Unknown in a session = blank, never 0

## Self-improving — the tracker learns what the user adds

The tracker improves from the user's own additions, automatically:
- **New exercise added** → it joins their catalog and baselines (weight = given, or 0 / not set), appears in plans, logging and recommendations, and is remembered under the user's own name for it
- **User's own name for a known exercise** ("the leg push thing" = their Leg Press) → remember that alias for that user only; next time, recognise it without re-asking, but still show the photo once if the alias was first used ambiguously
- **New vague phrase resolved by photo pick** → that phrase joins that user's photo-check triggers, so the same words get the same careful check next time, not a silent guess
- **Repeated choice** (they always pick the Smith option, they always train a muscle a certain way) → surface it once as a noticed pattern and, if they confirm, apply it as their default going forward
- Nothing learned from one user ever carries into another user's tracker

## Help me choose — when the user is not sure what to add or do

If the user is unsure which exercise to add, or which to do:
1. Ask one short question: what do you want to train, or what equipment is in front of you / available?
2. **Briefly describe** 2-4 candidate exercises: one line each on what it works and the machine or setup it uses. Keep it short, no lectures
3. **Recommend one** based on the user's own description (muscle wanted, equipment available, what is Ready on their heat map), and say why in one line. Label it a recommendation
4. Give the others as examples they can pick instead
5. They pick → then log or add it. A recommendation is never auto-logged, and if their pick is still machine-ambiguous, the Photo Rule applies before logging

## Calendar sync — one calendar, every logging channel

The tracker calendar is the single synced view, fed by every channel the user logs through:
- **Verbal / voice / chat**: a session the user reports is logged in the tracker and appears on the tracker calendar for that date
- **Calendar**: gym classes, PT bookings and training events on the user's calendar appear in the tracker calendar as scheduled sessions
- **Email**: booking confirmations, reschedules and cancellations received by email update the tracker calendar to match (booked = scheduled, moved = moved, cancelled = removed / marked cancelled)
- **De-duplication**: channels feed one record. Match on date + session/exercise name; if a possible duplicate is unclear, ask the user, do not create a second copy
- **Accuracy gate**: anything arriving from calendar or email is **scheduled only**. Only the user's confirmation that they attended makes it completed, and only completed sessions drive the heat map and recommendations

## Extensibility — the list is a starting set, never a cap

The exercise catalog and a user's baseline list are starting sets only. There is no limit on exercises or baselines.
- When a user logs an exercise that is not in their baselines yet, add it as a new baseline entry: weight = what they gave in that session, or **blank** if they gave none
- When a user names a new exercise during onboarding or later, add it — never refuse it because it is "not in the list"
- The catalog grows the same way: a confirmed new exercise (with its muscle mapping confirmed by the user, and its machine photo if one is needed) joins the catalog for that user

## Data model (Space)

Tables: `baselines` (group, exercise, weightLabel, note), `workout_plans` (name, focus, dayLabel, notes, exercises[]), `sessions` (date, title, type, status, timeLabel, notes, exercises[]), `macro_target` (calories, proteinG, carbsG, fatG, waterMl, notes — all nullable), `macro_logs` (date + same fields, all nullable), `muscle_states` (manual overrides only), `notes`.
Exercise item: `{ name, sets, reps, weight, note }` — sets/reps/weight are strings so "blank", units and ranges like "8-12" survive intact.
Dates are the user's local calendar days (YYYY-MM-DD) in their timezone from onboarding.

## Deliverable in this SKU

- This SKILL.md
- `onboarding.md` — the full question script, word for word
- `guardrails.md` — the one-page rule card
- `catalog.md` — exercise catalog, muscle map, look-alike photo pairs, recommendation catalog
- `README.md` — what it is, what it tracks, how it tracks, for sharing/marketing
