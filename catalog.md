# Catalog — Exercises, Muscles, Photo Pairs, Recommendations

**This catalog is a starting set, not a limit.** Users add new exercises as they log them; a new exercise creates a new baseline (weight blank unless the user gave one). Nothing is capped at the number of exercises listed here.

**Shipped-size note:** the named starting set below is what this package specifies. Larger libraries exist in grown installs — a library grows per user through the extensibility rule, so an install's exercise count is an instance result, not a shipped requirement of this catalogue. Do not state any grown install's count as the shipped catalogue size.

## Muscle groups (18, professional names)

Pectorals · Front Deltoids · Side Deltoids · Rear Deltoids · Trapezius · Upper Back · Lats · Lower Back · Biceps · Triceps · Forearms · Abs · Obliques · Glutes · Quads · Hamstrings · Adductors · Calves

## Exercise → muscle mapping (exact order — order is the rule)

Keyword based, in this exact order. The checks marked BEFORE exist because matching a generic word first credits the wrong muscle:

1. Leg Extension → Quads (primary)
2. Leg Curl → Hamstrings (primary). Check BEFORE generic "curl", or leg curl becomes Biceps.
3. Wrist Curl → Forearms. Check BEFORE generic "curl", or wrist curl becomes Biceps.
4. Curl (other) → Biceps; Hammer Curl also → Forearms.
5. Rear Delt Fly / Reverse Fly / Reverse Pec Deck → Rear Deltoids. Check BEFORE generic "fly / pec deck", or reverse pec deck becomes Pectorals.
6. Chest / Bench / Fly / Pec Deck → Pectorals. ("Press" or "bench" alone is ambiguous for machine ID — Photo Rule — even though the muscle is Pectorals.)
7. Lat Pulldown / Pull-up / Chin-up → Lats. Pulldown does NOT mark Biceps sore.
8. Row → Lats + Upper Back (primary); Lower Back + Trapezius (secondary).
9. Deadlift → Lower Back + Glutes + Hamstrings + Trapezius.
10. RDL / Romanian → Hamstrings + Glutes + Lower Back.
11. Back Extension → Lower Back.
12. Shrug → Trapezius.
13. Tricep / Pushdown / Skull Crusher / Dip → Triceps.
14. Shoulder Press / Overhead / Military Press → Front Deltoids + Side Deltoids.
15. Lateral Raise → Side Deltoids; Front Raise → Front Deltoids.
16. Face Pull → Rear Deltoids + Trapezius.
17. Squat / Leg Press / Lunge / Step-up → Quads + Glutes (primary); Hamstrings + Adductors (secondary).
18. Hip Adductor → Adductors. Hip Abductor → Glutes. Opposites — never swap them.
19. Hip Thrust → Glutes.
20. Sled Push → Quads + Glutes + Calves.
21. Bike / Cycling / Assault Bike → Quads + Glutes + Calves.
22. Calf Raise → Calves.
23. Sit-up / Crunch / Leg Raise / Knee Raise / Captain's Chair / Plank → Abs.
24. Twist / Oblique / Woodchop → Obliques + Abs (secondary).

Unmapped exercise name → ask the user which muscle it worked / show photos. Do not force-map it by guessing.

The full mapping also lives in `SKILL.md`; the two must never drift. If they ever disagree, that is a defect to fix, not a choice to pick from.

## Baseline groups and the onboarding exercise list

Baselines are grouped Push / Pull / Biceps / Lower Body / Core / Misc / Cardio. Onboarding (`onboarding.md` Block 6) walks the common exercises group by group:

- **Push:** Machine Chest Press, Incline Dumbbell Chest Press, Flat Dumbbell Bench Press, Smith Machine Bench Press, Shoulder Press, Lateral Raise, Tricep Rope Pushdown
- **Pull:** Lat Pulldown, Seated / Low Row, T-Bar Row, Machine Row, Pull-up (assisted or unassisted — ask)
- **Biceps:** Seated Dumbbell Curl, Barbell Curl, Hammer Curl, Machine Curl, Cable Curl
- **Lower:** Squat (Smith, barbell, or machine? — photo check), Leg Press vs Hack Squat (photo check), Leg Extension, Leg Curl, Hip Adductor vs Abductor (photo check), Romanian Deadlift / Deadlift, Calf Raise, Hip Thrust
- **Core:** bodyweight core work, as the user names it
- **Cardio:** bike, treadmill, assault bike, walks — distance/time only if the user knows it

Per exercise the user gives one of three answers: the weight (recorded exactly, with unit), "I do it but don't know the weight" (exercise recorded, weight blank), or "I don't do that one" (skipped, not added).

## Look-alike photo pairs (always show all options, labelled)

1. Smith machine bench press ↔ Free barbell bench press
2. Smith machine squat ↔ Barbell back squat
3. Smith machine shoulder press ↔ Machine (stack) shoulder press ↔ Dumbbell shoulder press (3-way)
4. Hack squat ↔ Leg press
5. Chest-supported T-Bar row ↔ Seated cable row ↔ Smith row ↔ Machine row
6. Hip adductor (pads push in) ↔ Hip abductor (pads push out)
7. Leg extension ↔ Leg curl
8. Close-grip Smith bench (Triceps) ↔ Normal Smith bench (Pectorals)

Also trigger the photo check on vague language: "the press", "a row machine", "the leg machine", "chest thing", "some Smith thing", "low row, maybe on the Smith". Hip adductor vs abductor is always photo-checked on first mention.

**Photo sourcing rule for implementers:** the shipped source is **`photos/` in this package** — the verified generic photo set (18 photos) with its index `photos/photos.md`, which lists every file, its look-alike pair, its telling feature and its source. An install copies `package/photos/` into its Space at setup so the photos always load. Use a real photo of the actual machine/movement, visually verified. Never use an illustration/render as a machine ID photo. If unverified, ship no photo rather than a wrong one. A photo verified for one movement is never reused as the identification photo for a different movement. Coverage against §"Look-alike photo pairs": all 8 pairs are covered by the shipped set (pair 3 has all three options; pair 5 has all four).

## Recommendation catalog (by ready muscle)

- Pectorals: Smith Bench Press / Machine Chest Press / Incline DB Press / Cable Fly
- Front Delts: Smith Shoulder Press / DB Shoulder Press
- Side Delts: Seated DB Lateral Raise / Cable Lateral Raise
- Rear Delts: Cable Rear-Delt Fly / Reverse Pec Deck
- Trapezius: Smith Shrug / DB Shrug
- Upper Back: Smith Row / Machine Row / Double Pulley Row
- Lats: Lat Pulldown / Straight-Arm Pulldown
- Lower Back: Deadlift / Back Extension
- Biceps: Seated DB Curl / Cable Curl / Barbell Curl
- Triceps: Rope Pushdown / Smith Close-Grip Bench
- Forearms: Hammer Curl / Wrist Curl
- Abs: Incline Sit-up / Captain's Chair
- Obliques: Medicine Ball Twist / Cable Woodchop
- Glutes: Hip Abductor / Smith Hip Thrust
- Quads: Smith Squat / Leg Extension / Leg Press
- Hamstrings: Leg Curl / RDL
- Adductors: Hip Adductor
- Calves: Smith Calf Raise / Barbell Calf Raise

**Recommendation engine:** Ready muscles only, max 6 exercises, priority order in `SKILL.md`, weight line = the user's baseline or "No baseline yet". Session title derived from the mix: Push / Pull / Leg / Upper-body / Full-body. A recommendation is never auto-logged.

## Starter plans (templates, weights all blank until onboarding fills them)

- **Upper Body:** chest press movement, lat pulldown, shoulder press, row, bicep curl, tricep pushdown
- **Lower Body:** squat movement, hinge (RDL/deadlift), leg press or hack squat, leg curl, leg extension, calf raise

Both templates use the user's confirmed scheme, or a blank scheme if they set none.
