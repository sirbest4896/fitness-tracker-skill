# Catalog — Exercises, Muscles, Photo Pairs, Recommendations

**This catalog is a starting set, not a limit.** Users add new exercises as they log them; a new exercise creates a new baseline (weight blank unless the user gave one). Nothing is capped at the number of exercises listed here.

## Muscle groups (18, professional names)
Pectorals · Front Deltoids · Side Deltoids · Rear Deltoids · Trapezius · Upper Back · Lats · Lower Back · Biceps · Triceps · Forearms · Abs · Obliques · Glutes · Quads · Hamstrings · Adductors · Calves

Full exercise → primary/secondary muscle map: see SKILL.md "Exercise → muscle mapping". Mapping order matters (Leg Curl before Curl; Adductor ≠ Abductor).

## Look-alike photo pairs (always show both, labelled)
1. Smith machine bench press ↔ Free barbell bench press
2. Smith machine squat ↔ Barbell back squat
3. Smith machine shoulder press ↔ Machine shoulder press ↔ Dumbbell shoulder press (3-way)
4. Hack squat ↔ Leg press
5. Chest-supported T-Bar row ↔ Seated cable row ↔ Smith row ↔ Machine row
6. Hip adductor ↔ Hip abductor
7. Leg extension ↔ Leg curl
8. Smith close-grip bench (Triceps) ↔ Smith bench (Pectorals)

Photo sourcing rule for implementers: use a real photo of the actual machine/movement, visually verify it, store it inside the Space so it always loads. Never use an illustration/render as a machine ID photo. If unverified, ship no photo. Verified generic photos for every pair above now ship in the bundled `photos/` folder (indexed in `photos/photos.md` with sources); an install should copy them into its Space so they always load.

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

Recommendation engine: Ready muscles only, max 6 exercises, priority order in SKILL.md, weight line = user's baseline or "No baseline yet". Session title derived from mix: Push / Pull / Leg / Upper-body / Full-body.

## Starter plans (templates, weights all blank until onboarding fills them)
- Upper Body: chest press movement, lat pulldown, shoulder press, row, bicep curl, tricep pushdown
- Lower Body: squat movement, hinge (RDL/deadlift), leg press or hack squat, leg curl, leg extension, calf raise
Both templates use the user's confirmed scheme, or blank scheme if they set none.
