# Guardrails Card — BestFit

Print this with the skill. These override convenience, speed and helpfulness.
**Rule zero: LOGGING ACCURACY is the most important thing in this skill. A wrong record is worse than a missing one. A blank is a correct answer; a guessed number is a wrong answer, even if it happens to be close.**

## The original rules (carried forward)

| # | Rule | What happens instead |
|---|------|----------------------|
| 1 | Unknown weight / reps / target / machine bar weight | Leave **blank**. Never 0, never "typical", never another user's number |
| 2 | Ambiguous machine or movement | Show look-alike **photos**, ask "Which one is it?", wait. Log nothing specific until they pick |
| 3 | Vague words: "the press", "a row", "leg machine", "chest thing" | Same as #2. Vague = photos |
| 4 | "Press" / "bench" without a setup | Muscle may be clear (Pectorals), machine is NOT. Photo check before naming the machine |
| 5 | Hip adductor vs abductor | Always photo-check on first mention. Opposite movements, opposite muscles |
| 6 | Leg curl | Maps to Hamstrings, never Biceps. Mapping order is fixed in SKILL.md / catalog.md |
| 7 | Lat pulldown | Marks Lats only. Does not make Biceps sore |
| 8 | Planned / mentioned workout | Status = scheduled. Only a confirmed "I did it" becomes completed, and only completed drives the heat map |
| 9 | Macro target the user does not have | Stays null/blank. A suggestion must be labelled "suggestion" and needs an explicit yes. A calculated target must be shown in full, labelled calculated, and needs an explicit yes |
| 10 | Food estimate | Only with prior consent, always labelled "estimate", never silently overwritten, user's own number always wins and replaces it. Enforcement split (stated honestly, per `framework.md` §F5; Gate G3 note, observation O-e): the estimates-permission gate is enforced in the **chat flow**, as a process rule — the app layer does not independently refuse an estimated write, and the product must not claim it does |
| 11 | Progression | Suggest + small only after all sets hit the top of the user's rep range with clean form, and only as a question. Never auto-increase a baseline |
| 12 | Units | Store as given (kg or lb), label it. No silent conversion |
| 13 | Pain / injury mentioned | Record as stated, recommend a qualified professional, do not diagnose or prescribe rehab/max loads |
| 14 | Data isolation | No user's data ever seeds another user's tracker. Every install starts empty |
| 15 | No photo available / no confident match | Show **no** photo rather than a wrong one. Ask for a description or a photo of their machine |
| 16 | Template default weight is 0 | 0 means "not set yet" only. Never a logged weight, never a recommended starting weight; logic treats 0 exactly like blank. And in a logged session, an unknown weight is blank — never 0 |
| 17 | User adds a new exercise / name / phrase | Learn it for that user only: catalogue + baselines update, alias and photo-trigger remembered. Never carried to another user |
| 18 | User unsure what to add or do | Briefly describe 2–4 candidates + their machines, recommend one from the user's own description, give the rest as examples. They pick; nothing is auto-logged |
| 19 | Session arrives via calendar or email | It enters the tracker calendar as scheduled only, under the sync contract (rules 20–24). Completed requires the user's own confirmation |

## The framework rules (F1 / F2 / F3 language — new in this version)

| # | Rule | What happens instead |
|---|------|----------------------|
| 20 | Sync matching (F1) | Match in this order only: externalRef → uid → fuzzy key. Never match on date + name alone — that was the reschedule-duplication defect. An unresolved fuzzy match goes to the ambiguity queue and the user is asked; it is never auto-merged and never silently duplicated |
| 21 | Sync writes (F1) | Sync may update source-owned fields only (date, time, title, scheduled/cancelled). It never overwrites user-owned fields (logged exercises, weights, corrections, completed status). A re-sync carrying an empty exercise list can never blank a logged session |
| 22 | Deletes and replays (F1) | Deletes are tombstones, not hard deletes; a stale source cannot resurrect a cancelled record. Every mutation carries an idempotency key; a replay is a no-op, never a second record |
| 23 | Provenance (F2) | Every macro log and session field group carries source, `provStatus` and source reference. `provStatus` is exactly one of: **observed** (user-given / synced as-is), **estimated** (an estimate, only with consent), **derived** (calculated by the tracker, e.g. a calculated target). User-facing words map to it: "user-given" = observed, "calculated" = derived, "estimate" = estimated. A correction keeps the prior value in record history — the original is never destroyed |
| 24 | Day completeness (F3) | Every macro and training day is none / partial / complete. A food day is `complete` **only** when breakfast, lunch and dinner are all logged, or the user confirms the day (e.g. a meal was skipped) — complete by the fact of the logged meals, never by a job declaring it. The food-source refresh **never writes completeness** (it may compute and show which meals are present). The system **never closes or locks a day on its own**: a day with missing meals stays **open**, and missing meals are asked about **gently** — "What did you have for lunch?", never "Did you have lunch?" — ask and wait, ask again the next day if unanswered. Training-day completeness is **user-asserted only** — the system never sets a training day complete. The system flags suspected-partial days, with a reason, and asks. A weekly summary flags any day with unlogged meals as not complete, naming what is missing. Averages and hit rates are computed over **complete days only** and are always printed with their coverage (complete days counted, partial days excluded, days not logged). A partial day is shown labelled partial; a blank day is skipped, never zero-filled |
| 25 | Confirm before save (F5) | Parse → proposed record → one material clarification if needed → confirmation bound to that exact revision → atomic save → read-back receipt. Save only on an unambiguous yes to: "Ready for me to save this? Reply yes to save, or tell me what to change." A yes to any other question is never consent. Any edit voids the earlier confirmation. **Nothing is logged unless the user confirms it:** if the agent is unsure (a machine pick is the main case), it asks in chat first in confirmation form ("Is this the one?"); if the user does not reply, the item stays unlogged and unlocked, and the pending confirmation takes the **last position** in that night's Ask Queue, asking again; still unconfirmed, it stays unlogged — it never expires into a guess. (The retry half is enforced in the app; the edit-voids-confirmation half is a chat-flow rule — state it that way.) |
| 26 | Recommendations never write | The Recommendations block, the Today steer and the morning brief are display/advice only. Stored targets change only on the user's Apply + Confirm, their own new number, or an accepted onboarding calculation. A recommendation is never auto-applied and never auto-logged |
| 27 | Automation consent | No job runs without its own consent record — all seven rows: ask queue, morning brief, calendar booking, email booking, macro refresh, steps sync, reconciliation. Each consent is separate (calendar booking sync and email booking sync are different answers; steps sync consent is a different answer from the steps source pick). A job switched on with no time chosen is do-not-send. No job ships with a default clock time, a default chat, or a default source (the booking mirror's interval is a duration parameter, confirmed by the user at onboarding) |
| 28 | Steps and weight writes | A day with no step data stays blank — never 0, never estimated. A steps write preserves the same date's weight; a weight write preserves the same date's steps |

**Precision principle:** a blank is a correct answer. A guessed number is a wrong answer, even if it happens to be close.
