# BestFit — an all-in-one fitness tracker Muse Skill, built as a Space

**Training + macros/food + steps, in one tracker.** BestFit is a framework, not a fixed app: everything is customisable — theme and colours (default dark blue, black and red), targets, goal mode, exercise catalogue, meal structure, and every automation job and its time — during onboarding and at any time after, just by telling the agent. Every default is a starting point; nothing is locked.

## What it tracks
- **Workouts** — sessions, plans and a calendar, with sets, reps and weights per exercise, plus set-level records underneath each session. Don't know a weight? It stays blank. Nothing is ever invented.
- **Baselines** — your current working weight for every exercise you actually do; the catalogue grows as you log new exercises.
- **Body heat map** — front and back figures, 18 muscle groups in professional names, coloured sore / recovering / ready, updated automatically from your logged sessions only.
- **Recommended workout** — built only from muscles that are ready, max 6 exercises, each with a verified real photo of the machine.
- **Macros** — daily targets and daily logs for calories, protein, carbs, fat and water, next to your training. A food day is only ever complete when breakfast, lunch and dinner are logged, or when you confirm the day yourself — the tracker never closes a day on its own.
- **Steps and weight** — a body dashboard with daily steps, weight against your own target, and a 30-day weight trend.
- **Goals** — your goal and what you want from the tracker, in your own words, plus an optional goal mode (Lose Fat, Recomposition, Gain Muscle, Gain Weight) that drives your daily recommendations.
- **Today dashboard** — a 3-point summary, your numbers against target, week-on-week comparison, and one priority recommendation.

## How it tracks
You just tell Muse in chat after a workout or a meal: what you did, and the numbers if you know them.
- The heat map updates itself from completed sessions: freshly worked = sore, then recovering, then ready again in about 3 days.
- Every exercise shows a photo of the machine. **Not sure which machine? Muse shows you labelled photos and asks "is this the one?" — it never guesses.** Nothing is logged unless you confirm it; if you're not sure and don't reply, it asks again in the evening list — it is never logged on a guess.
- Sync never duplicates: the same booking re-sent updates the same record, a cancelled booking stays cancelled until a genuinely newer re-booking restores it with its history.
- Optional automation, each job separately opt-in at your own time: a morning brief, an evening ask list (missing meals asked gently, by name), a macro refresh, a steps sync, and a weekly reconciliation.

## Onboarding
Onboarding is personal and runs in this order: the automation first (each job explained, each one your choice), then what to connect (food source, health platform for steps and weight, calendar, email), then you — your goals in your own words and what you want from this tracker — then what BestFit can do. It ships with a packaged onboarding prompt (`onboarding-prompt.md`) that you personalise yourself first, including your own meals, so the setup is shaped around your life. Anything you skip stays blank and is asked about later, never filled in by assumption.

## Inside the bundle
SKILL.md (the skill) · PRD.md (full product requirements and acceptance criteria) · design-specs.md (screens, charts, layout) · framework.md (the full data framework: sync, provenance, completeness) · automation.md (job templates) · onboarding.md (the full onboarding) · onboarding-prompt.md (the personalisable onboarding prompt) · catalog.md (exercise catalogue, muscle map, look-alike pairs) · guardrails.md (the no-guessing rule card) · DECISIONS.md (design decisions) · TEST-REPORT.md (how this version was tested: a from-scratch replica built from this package alone, gate-checked with mock data only) · photos/ (verified real machine photos).

Ships with zero personal data. Every user's tracker starts empty and only ever contains their own numbers.

**Download bestfit-v3-skill.zip from the v3 release, unzip it, and ask your Muse to install the BestFit Skill.**
