# Design Specs — BestFit

**Scope of this spec:** the five sections of the Space, their components and order, the charts, the heat map, and the photo rules. Layout, tabs and colours of the sections are a settled design: five sections, in order, nothing added and nothing renamed.
**Not in this spec:** any social-media or sharing card format. That is a personal extra of one install and is out of the generic product (see `PRD.md` §10). There is deliberately no such section below.

---

## 1. Global rules

- **Sections, in order:** 1. Today · 2. Train · 3. Calendar · 4. Macros · 5. More.
- **Dates** are the user's local calendar days (YYYY-MM-DD) in their onboarding timezone.
- **Units** are shown as stored, labelled (kg or lb for weights; grams for macros; steps as counts). No silent conversion.
- **Blank renders as blank.** A missing value shows as blank / "not logged" / "No baseline yet" as appropriate to the component. It never renders as 0, never as a fabricated value, and a gap in a chart is a gap, not a zero bar filled in silently (chart rules below state each chart's gap treatment).
- **Coverage labels.** Any average, hit rate or multi-day figure is printed with its coverage: the count of complete days it is over, partial days excluded, and days not logged. Format pattern: "over N complete days; P partial days excluded; M days not logged".
- **States and colours (muscle states only):** Sore = red, Recovering = amber, Ready = green. These three colours are the muscle-state palette everywhere the heat map or muscle state appears.
- **Design identity (pinned, product owner's design decision 14 Nov 2026 — supersedes the Gate G2 volt-lime identity; an implementer must not invent a different theme):** the theme is **dark blue, black and red**. It governs the chrome only — background, surfaces, navigation, headings, borders and accents. Exact palette, no substitutions:
  - **Background:** near-black `#04060B` (blue-black, never a warm or green-black).
  - **Surface / card:** deep navy `#0B1526`. Raised or nested surface: `#111F36`. Borders and dividers: `#22354F`.
  - **Dark blue primary:** `#1B4B8F` — primary buttons, links, selected controls and other main interactive elements. Pressed / active state of a primary element: `#2F6FD0`.
  - **Navigation:** bottom navigation bar with the five sections in the §1 order, each with a sub-label. Not a top pill bar. The bar itself sits on `#081120`; the **active tab is marked in the red accent** (label and indicator), inactive tabs in the secondary text colour.
  - **Red accent:** `#FF3B47` — highlights, the active tab, key numbers and calls to action. The red accent is **chrome only**: it is never used as a data fill (see the data-colour separation rule below).
  - **Text:** primary `#F2F5FA`; secondary `#9AA9C2` — both on the near-black background.
  - **Headings:** Barlow Condensed (display headings large and condensed); body type Barlow family. (Type is unchanged by the theme decision.)
- **Data colours are separate from the theme, and do not change with it.** Two functional palettes stay exactly as specified, because they carry meaning a user reads at a glance:
  - **Muscle-state palette:** Sore red `#C8102E`, Recovering amber `#E9A23B`, Ready green `#2FA84F` — used only as the soft gradient fills of §6 and their legend.
  - **Macro colour coding (everywhere macros are shown — target, breakdown, charts):** protein teal `#14919B`, carbs amber `#E9A23B`, fat blue `#3A6EA5`; over-benchmark values may be shown in red.
  - **Separation rule (accent red vs Sore red):** the red accent `#FF3B47` and the Sore fill `#C8102E` are different reds doing different jobs. The accent appears only on chrome (tab indicator, headings' highlights, key numbers, buttons) and **never inside the body figures**; a muscle is red only as a labelled Sore gradient fill with the legend beside it. No element may use the accent red in a position where it could be read as a muscle state, and no muscle fill may use the accent hex.
- **Photos** are real photos of the actual machine or movement, visually verified, stored inside the Space so they always load. The shipped source is `photos/` in the skill package (verified generic set + index `photos/photos.md`), copied into the Space at setup. Never an illustration or render as a machine-identification photo. If no verified photo exists, no photo is shown.

## 2. Today

The Today section is a daily dashboard. Components appear in this order, top to bottom:

1. **3-point executive summary.** Exactly three points. Each point is derived from the user's own logged data for today/yesterday/the recent complete days. If a point has no data behind it, it says so plainly rather than filling the slot with a generic line.
2. **Summary table.** Columns: **Today / Target / Yesterday / 7-day average**. Rows cover the tracked daily measures (at minimum: calories, protein, carbs, fat, steps, weight where logged). Target comes from the user's stored target record; a blank target renders blank. The 7-day average column carries the coverage label of FR-TODAY-2 / §1 of this spec.
3. **Stacked-bar macros chart.** Window: **the last 7 days**. One bar per day in the window, each bar stacked by macro composition (protein / carbs / fat shares of the day), in the §1 macro colours. Partial days are visually distinguishable and labelled partial; unlogged days are gaps, not zero-height bars presented as logged days. The chart's figures follow the complete-days-only rule for any average line or annotation drawn on it.
4. **Steps bar chart.** Window: **the last 7 days**. One bar per day for the window. A day with no step data is a gap labelled as not logged — never a 0 bar.
5. **Weight line chart (30-day window).** The trend line follows the EWMA-lite rule decided for this product (`DECISIONS.md`): exponential smoothing, alpha 0.3, over logged weigh-ins, computed over full logged history and displayed for the last 30 days. The line **breaks at gaps** in logging; it does not join across a gap. Weigh-in points may be shown under the trend line. With fewer than two weigh-ins, no trend is drawn; the component shows the logged value(s) and a plain "not enough data for a trend" state.
6. **Week-on-week table.** This week vs last week for the same daily measures as the summary table, **plus sessions completed**, with averages carrying coverage labels on both sides. A week with no complete days shows that honestly instead of a number.
7. **ONE priority recommendation.** Singular. Selected by the recommendation priority in `automation.md` (the same priority the morning brief uses), following the user's current goal mode. It states its basis (the number that caused it) in the same component. If nothing warrants a recommendation, it shows an honest on-track line — it never manufactures a problem to fill the slot.

**Data composition:** the section is served by a single dashboard read where available; otherwise it is composed from the macro summary, steps summary, weight summary, recommendations and overview reads (action names in `framework.md`).

**Empty-install state:** on a fresh install every component renders its honest empty state: no summary points invented, table cells blank, charts empty with "nothing logged yet", and the recommendation slot showing the log-first state rather than advice.

## 3. Train

Components:

- **Sessions list.** Each session: date, title/type, status (scheduled / completed / skipped), and its exercises (name, sets, reps, weight, note; unknown fields blank).
- **Plans.** Reusable plans built from the user's baselines, plus the two starter templates (Upper Body, Lower Body — compositions in `catalog.md`), weights blank until onboarding/logging fills them.
- **Baselines.** Grouped Push / Pull / Biceps / Lower Body / Core / Misc / Cardio. A baseline with no weight shows "No baseline yet" (a seeded 0 reads the same way — see the template rule in `SKILL.md`).
- **Exercise library.** A browsable catalogue. Each exercise entry carries its name, its machine/setup, and its verified real photo where a photo is needed for identification. The library is a starting set and grows per user (extensibility rule in `SKILL.md`); there is no fixed count requirement in the design.
- **Body heat map.** See §6.
- **Recommended workout.** Max 6 exercises, Ready muscles only, priority order per `SKILL.md`. Each recommended exercise shows: exercise name, machine/setup name, its photo, and a weight line that is either the user's baseline or "No baseline yet — leave blank / build up by feel". The session title is derived from the mix by the rule in `PRD.md` FR-TRAIN-8: a category (Leg / Push / Pull) holding more than half the exercises gives its title; no majority with Leg plus upper-body present = Full-body; upper-body-only mix = Upper-body.

## 4. Calendar

- **One calendar view** holding planned (scheduled) and completed sessions together; skipped sessions are distinguishable.
- Each entry shows date, time where known, title, and status. A scheduled entry is never styled or counted as a completed workout.
- A cancelled booking's record is tombstoned: it does not appear as an active entry, and its history remains available in the record history / sync log surfaces (More / framework status), not as a calendar entry.
- A rescheduled booking appears once, on its new date. There is never a pair of entries for one booking.
- **No demo furniture:** the Calendar section is the calendar view only. No "try the sync path" demo form or other test scaffolding ships in the product; the booking-mirror behaviour lives entirely in the sync actions (`framework.md`, `automation.md` Job 7).

## 5. Macros

- **Target display.** The stored daily target (calories, protein, carbs, fat, water, notes) with blank fields shown blank. The target and the daily macro breakdown are **colour-coded** in the §1 macro colours (protein teal, carbs amber, fat blue) — a plain uncoloured definition list does not satisfy this spec. Where a target was calculated and accepted, it is labelled calculated; where user-given, it is the user's own — the display distinguishes the two via the provenance label.
- **Daily logs.** One row/record per date with the same fields; missing values blank. A partial day is labelled **partial** wherever the day is shown (day view and lists), and a day that is not complete is shown as **open** — never locked or closed by the system. A suspected-partial day shows its flag and reason until the user resolves it. A day displays **complete** only when breakfast, lunch and dinner are all logged or the user confirmed the day (see `framework.md` §F3); the food-source refresh never sets the state, and the user can complete or correct an open day at any later time.
- **Past-7-days chart.** The recent-days macro view. Its averages follow the complete-days-only rule with the coverage label printed at the chart (§1).
- **Body dashboard (within Macros):**
  - **Steps block.** Recent daily step counts, with the source labelled (the user's chosen source). No-data days blank.
  - **Weight block.** Recent weights and, if the user set a target weight, the weight shown against that target. The trend display follows the same EWMA-lite + gap-break rule as the Today weight chart.
  - **Recommendations block.** Shows the computed recommendation (maintenance basis, goal mode, preset, protein/fat slider values, resulting protein/fat/carbs) next to the stored target, clearly display-only, with an **Apply** action that requires a following **Confirm** before any stored target changes. States:
    - No weight logged → "log your weight first", nothing computed.
    - Missing another required input → names the missing input, nothing computed, targets untouched.
    - Carbs clamped at 0 → the clamp is visible in the shown working, not hidden.

## 6. Body heat map (component spec)

- **Muscles (18, professional names only):** Pectorals, Front Deltoids, Side Deltoids, Rear Deltoids, Trapezius, Upper Back, Lats, Lower Back, Biceps, Triceps, Forearms, Abs, Obliques, Glutes, Quads, Hamstrings, Adductors, Calves.
- **Figures:** front and back body figures, soft gradient rendering — **not square blocks**.
- **Silhouette (pinned, product owner's design decision 14 Nov 2026):** both figures are a **lean, athletic human silhouette** — defined shoulders, a clearly tapered waist, slim straight legs. The figures must **not** be wide, rounded or stocky: no oval torso blob, no broad midsection, no thick limbs. Proportions a builder can follow, the same for front and back:
  - Head-to-body ratio about **1:7** (the figure is about seven head-heights tall).
  - Shoulder width about **2.3 head-widths**; waist width about **1.35 head-widths** — a shoulder-to-waist ratio of roughly **1.7:1**, so the taper from shoulders to waist is unmistakable.
  - Hips narrower than the shoulders (about 1.6 head-widths); the torso outline runs as a taper (trapezoid), never a parallel-sided or bulging outline.
  - Legs slim and straight with a visible gap between them from thigh to ankle; arms slim, hanging straight at the sides, close to the body.
  - Muscle zones (§6 colours) sit inside this lean outline; a zone may not widen the silhouette.
- **Colours:** Sore = red `#C8102E`, Recovering = amber, Ready = green, applied as a soft gradient over each muscle's area. These are data fills (§1 separation rule): the theme's red accent never appears inside the figures.
- **Adductors geometry (hard rule):** on the front figure, the Adductors are drawn as **two slim inner-thigh strips, one on each leg**. They are never drawn as one single central blob between the legs. Any rendering that merges them into one central shape fails this spec.
- **State derivation:** automatic, from completed sessions only, under the cooldown rules in `SKILL.md` (primary: 0–1 days Sore, 2 days Recovering, 3+ Ready; secondary: 0–1 days Recovering, 2+ Ready).
- **Two layers on screen:** the data layer distinguishes never-logged ("no data yet", `hasHistory = false`) from recovered; the display layer renders never-logged muscles as Ready (green). The screen must not claim a never-logged muscle is "recovered". (Surfacing a distinct no-data visual is a future design decision, out of this spec.)
- **Manual override:** tapping a muscle sets an override. An override is visually the state the user set, is stored distinctly from session-derived state, wins only while newer than the last session that worked that muscle, and lapses after 3 days.

## 7. Exercise photo grid rules

- Photos appear in: the exercise library, the recommended workout, plan/session exercise rows where a photo aids identification, and the chat Photo Rule flow.
- **Grid presentation:** library and recommendation photos are presented in a grid of exercise cards; each card shows the photo, the exercise name, and the machine/setup name. A card with no verified photo shows the name and setup with no image — never a substitute image, never a render.
- **One exercise, one photo.** Each exercise uses its own verified real photo of that machine/movement. A photo verified for one movement is not reused as the identification photo for a different movement.
- **Full catalogue coverage (Gate G2 amendment).** Every exercise in the shipped catalogue carries a verified real photo, sourced at build/install time from generic external sources (retailer, manufacturer, exercise-guide, editorial or commons imagery of the actual machine/movement), visually verified, and stored inside the Space. An exercise without a verified photo is a build defect, not an acceptable state. The `photos/` set in the skill package remains the **mandatory look-alike disambiguation set**: every look-alike pair in `catalog.md` must be covered by it, and the Photo Rule flow uses it. Photos sourced at build time extend coverage beyond that set; they never replace it, and no personal or user-supplied image is ever used as a catalogue photo.
- **Look-alike presentation:** in the Photo Rule flow, all look-alike options are shown together, each labelled "✓ This one" / "✗ Not this one", and the user picks before anything specific is logged (pairs list in `catalog.md`).
- **Storage:** photos are stored inside the Space so they always load, independent of any external source staying online. The shipped set an install copies in is `photos/` in the skill package (index: `photos/photos.md`).

## 8. More

- More is the fifth section. Its evidenced content is minimal (see `PRD.md` FR-MORE-1/2): an implementation-defined settings and help area.
- It presents, in a simple list (no dashboard treatment): profile basics; the goals record and macro target state; the automation consent list with each job's on/off, chosen time and source; the user's notes; and a short help text (how to log, how to report food, what blank means, how to change a setting, and the charter restated in one line: everything in BestFit is customisable and changeable at any time through the agent).
- Changes from More are made through the agent with read-back and confirmation (F5); More itself holds no direct-edit forms in this version.
- More must not present any unimplemented feature (per-muscle set targets, meal detail, sleep sync, progression screens) as available.

## 9. Chat surfaces (design-relevant)

These are not Space sections, but they are designed surfaces of the product:

- **Save read-back / receipt:** after any confirmed save, the chat shows a short receipt of what was saved (the record and its key fields), matching the F5 transaction.
- **Ask Queue message:** one message, questions numbered, one line each, missing-food first (full spec in `automation.md`).
- **Morning brief message:** greeting, optional missing-meals chase, 3 summary points as dash bullets, one numbers line, one closing recommendation (full spec in `automation.md`).
- **Reconciliation report:** a short headed report: either "all counts match, no issues" or a numbered list of exact differences.
- None of these messages use generic filler content; every line is computed from the user's own logged data or states plainly that a source was unavailable.
