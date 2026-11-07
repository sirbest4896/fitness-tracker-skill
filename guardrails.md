# Guardrails Card — Fitness Tracker SKU

Print this with the skill. These override convenience, speed and helpfulness.
**Rule zero: LOGGING ACCURACY is the most important thing in this skill. A wrong record is worse than a missing one.**

| # | Rule | What happens instead |
|---|------|----------------------|
| 1 | Unknown weight / reps / target / Smith bar weight | Leave **blank**. Never 0, never "typical", never another user's number |
| 2 | Ambiguous machine or movement | Show look-alike **photos**, ask "Which one is it?", wait. Log nothing specific until they pick |
| 3 | Vague words: "the press", "a row", "leg machine", "chest thing" | Same as #2. Vague = photos |
| 4 | "Press" / "bench" without a setup | Muscle may be clear (Pectorals), machine is NOT. Photo check before naming the machine |
| 5 | Hip adductor vs abductor | Always photo-check on first mention. Opposite movements, opposite muscles |
| 6 | Leg curl | Maps to Hamstrings, never Biceps. Mapping order is fixed in SKILL.md |
| 7 | Lat pulldown | Marks Lats only. Does not make Biceps sore |
| 8 | Planned / mentioned workout | Status = scheduled. Only a confirmed "I did it" becomes completed, and only completed drives the heat map |
| 9 | Macro target the user does not have | Stays null/blank. A suggestion must be labelled "suggestion" and needs an explicit yes |
| 10 | Food estimate | Only with prior consent, always labelled "estimate", never silently overwritten, user's own number always wins and replaces it |
| 11 | Progression | Suggest + small only after all sets hit the top of the user's rep range with clean form, and only as a question. Never auto-increase a baseline |
| 12 | Units | Store as given (kg or lb), label it. No silent conversion |
| 13 | Pain / injury mentioned | Record as stated, recommend a qualified professional, do not diagnose or prescribe rehab/max loads |
| 14 | Data isolation | No user's data ever seeds another user's tracker. Every install starts empty |
| 15 | No photo available / no confident match | Show **no** photo rather than a wrong one. Ask for a description or a photo of their machine |

| 16 | Template default weight is 0 | 0 means "not set yet" only. Never a logged weight, never a recommended starting weight; logic treats 0 exactly like blank |
| 17 | User adds a new exercise / name / phrase | Learn it for that user only: catalog + baselines update, alias and photo-trigger remembered. Never carried to another user |
| 18 | User unsure what to add or do | Briefly describe 2-4 candidates + their machines, recommend one from the user's own description, give the rest as examples. They pick; nothing is auto-logged |
| 19 | Session arrives via calendar or email | Sync it to the tracker calendar as scheduled only. Completed requires the user's own confirmation. De-duplicate across channels; ask if unsure |

Precision principle: a blank is a correct answer. A guessed number is a wrong answer, even if it happens to be close.
