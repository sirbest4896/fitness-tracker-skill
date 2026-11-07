# A/B Test Report — Fitness Tracker SKU (v1 draft vs v2 final bundle)

Date: 2026-11-07 · Tester: Donna (Muse), self-onboarded · Method: simulated onboarding

## How the test was run — read this first
No external users were involved. I wrote two fictional personas, ran each one through the full onboarding + a first session log, and scored the saved record against a fixed 40-point checklist (20 points per persona). The same persona answers were fed to two rule sets:
- **A = V1**, a naive first draft of the skill (defaults and guessing allowed, the way a quick build would behave)
- **B = V2**, the final bundle in this folder (guardrails enforced)

Precision = checks passed / 40. A guessed or borrowed number counts as a FAIL even if plausible. A correct blank counts as a PASS. The runnable test is `test/ab_test.py`; raw results in `test/ab-results.json`.

Honest limit: 100% on V2 means the *rules as written* pass this checklist. It is not a live-user accuracy claim. Real users will invent new vague phrases the photo-check trigger list does not yet contain — the guardrail covers this by rule ("any vague name = photo check"), but the trigger word list should grow from real use.

## The personas
- **Persona A — experienced.** Trains with his own scheme (4 sets of 6-10), knows 5 baseline weights exactly, protein target known but calories unknown, Smith bar weight unknown, says "the press" in his session log, mentions a planned session for tomorrow.
- **Persona B — beginner.** Knows no weights at all (3 exercises, all unknown), no rep scheme and did not accept a template, wants macro tracking but has no targets yet, works in lb, describes machines as "the leg machine", "chest thing", "low row maybe smith", and says he *might* go Saturday.

## Results
| Rule set | Persona A | Persona B | Overall precision |
|----------|-----------|-----------|---------------------|
| V1 naive draft | 11/20 (55%) | 3/20 (15%) | **14/40 = 35.0%** |
| V2 final bundle | 20/20 (100%) | 20/20 (100%) | **40/40 = 100%** |

Beginners were hurt worst by the naive draft — exactly the users onboarding is for. Almost every V1 failure was an invented number or a guessed machine landing on the person who could not spot the error.

## Flaws found in V1, and the fix in V2 (9 flaw classes, 26 failed checks)
1. **Unknown weights defaulted to "0 kg"** (B01, B02, B03, B16) → Fixed: unknown = blank string. 0 is a false record, blank is the truth.
2. **Smith bar weight borrowed from an existing setup** (A08, B06) → Fixed: bar weight is asked per user; unknown = blank. This was also a personal-data leak risk — another user's constant must never ship in a generic SKU.
3. **Macro targets invented** — protein filled as a "typical" 150 g, calories as 2000 (A10, B07, B08, B17) → Fixed: targets stay null until the user gives them. Macros-on with all targets blank is a valid starting state.
4. **Rep scheme imposed** — 3 sets of 8-12 written over a beginner who never accepted it (B05) → Fixed: scheme is the user's, or a labelled template they explicitly accept, or blank. (Persona A's own 4x6-10 survived both versions.)
5. **Vague machines guessed** — "the press", "the leg machine", "chest thing", "low row maybe smith" all became "Smith Machine Bench Press" in V1 (A11, B09, B10, B11, B18, B19) → Fixed: Machine Photo Rule. Vague = show the look-alike photos, ask "Which one is it?", log nothing specific until they pick.
6. **Leg Curl mapped to Biceps** — naive keyword order hits "curl" first (A03, A19) → Fixed: mapping order locked, Leg Curl checked before Curl, maps to Hamstrings.
7. **Lat Pulldown marked Biceps sore** (A05) → Fixed: Pulldown = Lats only, so arms stay Ready after a back day, which is how the heat map recommendation is supposed to work.
8. **Hip Abductor treated as Hip Adductor** (A06) → Fixed: separate mappings (Abductor → Glutes, Adductor → Adductors) and a mandatory photo check on first mention — they are opposite movements.
9. **Planned sessions logged as completed** — "Legs tomorrow" and "Maybe gym Saturday" drove the heat map in V1 (A13, A14, B12, B20), plus **lb silently converted** (B13) → Fixed: mentioned/planned = scheduled only; only a confirmed session is completed; units stored as given and labelled.

## Privacy verification
Grepped the finished bundle for the source user's name, trainer, bar weight, protein target, body-weight goal, city, gym and account names: **0 hits**. Every install starts empty.

## What was NOT tested
- Live Space UI build (this SKU is the skill + onboarding + rules; building it as a fresh user's Space is the next step after approval)
- Real photo delivery in chat (photo pairs are specified and catalogued; the source Space holds 48 verified machine photos that can be reused as the generic photo library — they are equipment photos, no personal data)
- Food-photo macro estimation (out of scope for this bundle; the tracker logs macro numbers the user gives or agrees to have estimated and labelled)

## Addendum — v2.1 refinements (owner, 2026-11-07, after the main test)
Owner added four requirements after seeing the summary: (1) the tracker must be self-improving from what a user adds; (2) a fresh template's default weights are all zero; (3) if a user is unsure what to add or choose, briefly describe the exercise and machine, recommend from their description, and give examples; (4) logging accuracy is the most important rule; plus (5) calendar sync — sessions logged verbally, via calendar, or by email all sync into the one tracker calendar.
- Reconciliation on zero: template baselines default to 0, and 0 is defined in the skill as "not set yet" — treated exactly like blank by all logic, never a logged weight and never a recommended starting weight. The earlier blank rule still governs actual logging.
- Extra test: `test/ab_test_v21_addendum.py`, 10 checks covering zero-as-unset, new-exercise learning, user aliases, recommendation-never-auto-logged, and cross-channel de-duplication with scheduled-until-confirmed. Result: 10/10 (100%). Combined with the core test: 50/50.
