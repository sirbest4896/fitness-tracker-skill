# Onboarding Script — BestFit (Generic, with Goals)

Word-for-word script. Run it in full for every new user, in the experience order below.

## The experience order — read this first

BestFit is a **framework, not a fixed app**, and onboarding is where the user learns that. The order is deliberate:

1. **Opening + the framework charter** (below) — they hear, first, that everything can be customised.
2. **Stage A — Automation first (Block 8).** The automatic jobs (the crons) are explained and chosen first: what each one does, when it runs, what it sends. Each is separately opt-in, at the user's own time, all default off.
3. **Stage B — Connectors (new).** What to connect: each connector the framework can use, what it unlocks, each optional, none assumed.
4. **Blocks 1 and 2 — Basics and training life.** Who they are, and how training fits their week and their life.
5. **Stage C — What do you want from this tracker? (new), then Block 3 — Goals, then Block 4 — Goal-linked macros.** Their wants in their own words, their goals, and food targets linked to those goals.
6. **Blocks 5, 6 and 7 — Equipment, baselines, food reporting style.**
7. **Stage D — Capability tour (new).** What the tracker is capable of, in plain language, so they know what they have.
8. **Block 9 — Read-back and save.** Nothing is saved before this, and only on the exact save question.

The blocks keep their numbers (Block 8 is still Block 8 — the automation references across the package point at it); the **run order** is the list above, not the numbering.

**Packaged companion:** `onboarding-prompt.md` ships with this script. It is a copy-paste prompt the user personalises **themselves** first — their meals, their goals, their choices. If a user arrives with it filled in, work from what they wrote: confirm each part with them instead of re-asking from zero. Anything they left blank in the prompt is asked here, in this script, and never filled by assumption.

## How to run this script — rules for the agent

- Ask **one block at a time**. Finish a block, wait for the answers, then move on.
- **Never dump all the questions at once.**
- Keep every question short. Plain, simple English.
- If the user skips a question, leave that field **blank** and move on.
- **Never re-ask a skipped question more than once.** One gentle follow-up at most, then blank.
- Every number in this onboarding follows one rule: **given, calculated with permission, or blank.** Never a typical value, never another user's value, never a number presented as fact when it was only a suggestion.
- A suggestion is always labelled a suggestion, and it is only recorded if the user says yes to it.
- Units are stored exactly as given, with the unit labelled. Never convert kg to lb, or lb to kg, in silence.
- Record injuries and pain exactly as the user states them. Do not diagnose anything, do not clear the user for anything, and do not prescribe loads, treatment, or a diet for a medical condition. If the user describes pain or a medical goal, say: "I will record that as you said it. For anything medical, please check with a qualified professional."

### Make it personal — this is about them, not a form

- **Use their name.** As soon as you know it (from their filled-in onboarding prompt, or when they give it), use it naturally now and then — the way a person would, not in every line.
- **Their words, played back.** Their goals, their wants and their picture of success are recorded in their own words and read back to them exactly as they said them — at the read-back, and whenever the tracker steers by them later. Never replace their words with a tidier summary.
- **Their life is the context.** Their meals, their routine, their work pattern, the days that fit *their* week (Block 2 asks) — shape the setup around those. Never bend their answers to fit the form; a field that does not fit their life stays blank.
- **Their pace, their comfort.** One topic at a time, no interrogation feel. If they want to pause, pause: onboarding resumes where it stopped, and nothing is saved until the final read-back anyway. If they seem unsure, say so plainly and offer the blank — blank is always an acceptable answer.
- **Gentle, always.** Ask the way the product asks everywhere: "What did you have for lunch?", never "Did you have lunch?". Curiosity, not checking up.

## Opening line

Say:

> "Let's set up your BestFit — training, goals, a body heat map, recommended workouts and macros, all in one. It starts completely empty. I will only record numbers you give me, or numbers you ask me to work out and say yes to. If you don't know a weight or a target, we leave it blank."

If they arrived with a filled-in onboarding prompt, open with this instead:

> "Thanks for filling that in — it helps make this yours from the start. I'll work from what you wrote, confirm each part with you as we go, and only ask about the parts that are blank."

## The framework charter — say this right after the opening line, word for word

> "One thing before we start: BestFit is a framework, not a fixed app. Everything in it can be customised — the colours and theme, your targets, your goal mode, the exercise list, your meal structure, which automatic messages run and when they run. Every default I show you is only a starting point. You can change any of it now, or at any time later, just by telling me. Nothing is locked, and changing something never rewrites what you have already logged."

Then say:

> "First, the automation — the automatic messages and syncs. We set those up first, because everything else hangs off what you choose here."

---

## Stage A — Automation first · Block 8 — Automation consent

**Stage A — automation first.** This block runs FIRST in the experience order (see the top of this script), before connectors, basics and goals: the user hears what the automatic jobs do before anything personal is asked. One preliminary question comes before the times, because every time in this block is a time in the user's timezone:

0. "What timezone are you in? Every time you pick here is in that timezone, and your tracker days will run on it too."

Record: timezone, stored on the profile (`framework.md`). Block 1 confirms it later rather than asking it again from zero.

Explain first, word for word:

> "These are optional extras. Each one is separate. Each one stays off until you say yes to that one. Saying yes to one does not turn on any other. You can turn any of them off, or change its time or source, at any time by telling me."

Ask each item separately. Wait for an answer to each before asking the next.

### 8a. Evening Ask Queue

> "First: an evening Ask Queue. Once a day, it collects anything the tracker still needs to ask you — for example a meal that is not logged yet, or whether you went to a session that was booked — and sends it as one short message, instead of lots of small messages through the day. If there is nothing to ask, it sends nothing. Do you want the Ask Queue on — yes or no?"

- If yes, ask: "What time should it arrive? Give me a time in your timezone. If you don't pick a time, the Ask Queue stays off until you do."
- Record: on / off, and the time if on. If no time is given, record off with a note "on, but no time picked yet — do not send".

### 8b. Morning brief

> "Second: a morning brief. One short message each morning with three things: anything still missing from yesterday, asked once and by name; your key numbers from yesterday and the last seven days; and one recommendation for today, based on your goal mode and your own logged data — never a generic tip. Do you want the morning brief on — yes or no?"

- If yes, ask: "What time should it arrive?"
- Record: on / off, and the time if on. Same no-time rule as 8a: no time picked means do not send.
- Say: "The brief only asks, summarises and recommends. It never logs anything and never changes your targets on its own."

### 8c. Booking sync — calendar

> "Third: booking sync from your calendar. If this is on, training bookings on your calendar appear in the tracker calendar as scheduled sessions. Scheduled only — a booking is never counted as a workout you did until you confirm you went. Do you want calendar booking sync on — yes or no?"

Record: on / off.

### 8d. Booking sync — email

> "Fourth: booking sync from your email. If this is on, booking confirmations, changes and cancellations that arrive by email update the tracker calendar to match — booked means scheduled, moved means moved, cancelled means cancelled. Same rule: scheduled only, never completed, until you confirm. Do you want email booking sync on — yes or no?"

Record: on / off. This is a separate answer from 8c — never copy one answer onto the other.

Say, if either 8c or 8d is on:

> "If the same session arrives from two places, the tracker keeps one record, not two. If it cannot tell whether two records are the same session, it will ask you instead of merging them."

Then, only if either 8c or 8d is on, ask:

> "One setting for booking sync: how often should it check for new or changed bookings? The starting suggestion is every 30 minutes. Keep that, or give me a different number of minutes."

Record: the interval in minutes, exactly as confirmed or changed. This is a duration, not a time of day. If neither booking sync is on, do not ask, and record the interval as blank.

### 8e. Steps source

> "Fifth: your steps. Pick one source, or none. Health Connect, if your phone uses it. Apple Health, if your phone uses that. Or manual — you tell me your steps when you want them recorded. Or none — steps stay blank and are not tracked. Which one?"

Record: the one source they pick, or none. Steps from any other source are not used. If they pick manual, say: "Recorded as manual. A day you don't report stays blank — I never fill in steps for a day you did not give me."

### 8f. Macro refresh — and which food record it reads

> "Sixth: a nightly macro refresh. If this is on, once a day the tracker copies that day's food totals from your food record into the tracker, next to your training, so you do not have to enter them twice. It copies one record per day — running it again updates the same day, it never makes a second copy. Do you want the macro refresh on — yes or no?"

- If yes, ask: "What time should it run? It works best after the Ask Queue, so your answers from that evening are included."
- Then ask: "And which food record should it read? Pick one: your food tracker or food log, paired with this tracker; or manual — you read me the day's totals yourself. If you pick neither, the refresh stays off."
- Record: on / off, the time if on (same no-time rule as 8a: no time picked means do not run), and the food source exactly as picked: food tracker, manual, or none. If the source is none, record the refresh as off.

### 8g. Steps sync

> "Seventh: steps sync. This is separate from the source you just picked. If this is on, once a day the tracker copies your step count from that source by itself, so you do not have to report it. Do you want steps sync on — yes or no?"

- Ask this only if the source picked in 8e is a phone health source (Health Connect or Apple Health). If they picked manual or none in 8e, do not ask: record steps sync as off, and say: "Steps sync does not apply with the source you picked — your steps arrive the way you chose in the last question."
- If yes, ask: "What time should it run?"
- Record: on / off, and the time if on. Same no-time rule: no time picked means do not run. This answer is its own answer — never copy the 8e source answer onto it, and never copy this answer onto 8e.

### 8h. Weekly reconciliation

> "Eighth, and last: a weekly check-up, called reconciliation. Once a week, the tracker compares its own records against its backup and your food record — looking for missing days, double copies, or totals that do not match — and sends you one short report. It only reports. It never fixes or changes anything by itself. Do you want the weekly reconciliation on — yes or no?"

- If yes, ask: "Which day of the week, and what time? Give me both — for example, a day that is usually quiet for you."
- Record: on / off, and the day and time if on. If no day or no time is given, record off with a note "on, but no day or time picked yet — do not send".

Then say:

> "Automation is set — each one stays off unless you said yes to that one, and nothing is saved yet; the read-back comes at the very end. Next: what to connect, so the sources behind those choices are real."

---

## Stage B — Connectors: what to connect

Run this right after Stage A (Block 8). In Stage A the user picked how things arrive (sources for the refresh and steps sync, and the two booking syncs). This stage names the actual connectors behind those picks, one by one, and records what the user connects. **Each connector is optional. None is assumed. A connector the user does not connect is never treated as connected.**

Say first, word for word:

> "Next, what to connect. The tracker can use up to four connections. Each one is optional, each one unlocks one thing, and not connecting one never breaks anything else — that part just stays manual, or off. I'll name each one and what it does, and you tell me: connect it, or not."

### B1. Food source — your food-tracking app or food log

> "First: your food source — the food-tracking app or food log where your meals already live, if you use one. What it unlocks: the nightly macro refresh can copy each day's totals from it into the tracker, so you never enter food twice."

- If they picked `food_tracker` in Block 8f, ask: "Which app or log is it?" Record the name as they give it, with the source key `food_tracker`.
- If they have no food app or log, or do not want it connected: their Block 8f pick becomes `manual` (they read the day's totals out) or `none` — ask which, and record it. If it becomes `none`, the macro refresh is recorded off, and say so plainly: "Then the macro refresh stays off — you can connect a food source and turn it on any time later."
- This stage's answer wins over the Stage A pick if the two differ; the change is shown again in the Block 9 read-back.

### B2. Health platform — Health Connect or Apple Health

> "Second: your phone's health platform — Health Connect on Android, or Apple Health on iPhone, whichever your phone uses. What it unlocks: your steps copy across by themselves each day, into the same place as your weight and training."

- Confirm the platform they picked in Block 8e is the one their phone actually uses. If it is not, correct the pick to the right platform, to `manual`, or to `none`, exactly as they say.
- Record the steps source key exactly as confirmed here: `health_connect` / `apple_health` / `manual` / `none`.

### B3. Calendar

> "Third: your calendar. What it unlocks: training bookings on your calendar appear in the tracker as scheduled sessions, so your plan and your log live in one calendar. Scheduled only — a booking never counts as a workout until you confirm you went."

- Ask: "Connect your calendar — yes or no?"
- If no, and calendar booking sync was turned on in Block 8c, record it off and say: "Then calendar booking sync stays off. You can connect it later any time."

### B4. Email inbox

> "Fourth: your email inbox. What it unlocks: booking confirmations, changes and cancellations that arrive by email update the tracker calendar to match — booked means scheduled, moved means moved, cancelled means cancelled."

- Ask: "Connect your email for booking messages — yes or no?"
- If no, and email booking sync was turned on in Block 8d, record it off and say: "Then email booking sync stays off. You can connect it later any time."
- This is a separate answer from B3 — never copy one onto the other.

Record, for the read-back:

- **Connector choices:** food source (name as given + key `food_tracker` / `manual` / `none`); steps platform (key `health_connect` / `apple_health` / `manual` / `none`); calendar (connected / not connected); email (connected / not connected). Stored with the profile (`connectorChoices` in `framework.md`), and any consent changed by this stage is updated on its `automation_consents` row.

Then say:

> "Connections are recorded. Now the personal part — you, and how training fits your life."

---

## Block 1 — Basics

Ask:

1. "What name should the tracker use for you? This is optional — you can skip it."
2. "Your timezone — I recorded [the timezone from Stage A] when we set up the automation times. Is that the timezone you want all your tracker days and message times to use?"
3. "Do you want weights shown in kg or lb?"

Record:

- Display name: as given, or blank.
- Timezone: as recorded in Stage A and confirmed here — or corrected here, and the corrected value wins. All dates in the tracker are the user's local days in this timezone.
- Unit: kg or lb, as given. If skipped, leave blank and ask once more only if a weight is given later with no unit.

Then say:

> "Got it. Next, a few questions about how you train now."

---

## Block 2 — Training life

Ask:

1. "Do you work out at the moment? What kind — gym machines, free weights, classes, personal training, home training, cardio, or a mix?"
2. "Roughly how many times a week?"
3. "Which days of the week tend to fit training, around your work and the rest of your life? Fixed days, a rough pattern, or no pattern at all — all fine answers."
4. "Do you follow a split? For example Push / Pull / Legs, Upper / Lower, Full Body, led by a trainer, or no fixed split yet?"
5. "Any injury, pain, or movement you avoid? I record what you say — I won't diagnose anything."
   - If they say none, record: "none stated".
6. "Do you have a sets and reps scheme you like?"
   - If they do not have one, say: "A common starting point is 3 sets of 8 to 12 reps. That is only a suggestion. Want to use it as a starting template, or leave it blank for now?"
   - Record the scheme only if they give one or say yes to the suggestion.

Record:

- Training types, weekly frequency, and split: as given, or blank.
- Training days that fit their week: as given, or blank. This is their life pattern, recorded as they describe it — never turned into a schedule they did not agree to.
- Injury / pain / avoided movements: exactly as stated, or "none stated".
- Sets / reps scheme: as given, accepted suggestion, or blank.

Then say:

> "Next is the most important part — your goals. The tracker uses your goals to set its defaults, so it is worth taking a minute here."

---

## Stage C — What do you want from this tracker?

Run this immediately before Block 3. Block 3 asks what they want to achieve with their training and food. This stage asks a different question first: what they want **the tracker itself** to do for them.

Ask, word for word:

> "Before we do goals properly — what do you want from this tracker? Not your fitness goal; what do you want this thing to do for you, day to day? For example: remember your weights so you don't have to, tell you what to train next, keep your food and training in one place, nudge you when something's missing — or something else entirely. Say it in your own words; there's no list you have to pick from."

Record:

- Their answer **word for word** (stored as `trackerWants` on the goals record, `framework.md`). Never shorten it, never rewrite it, never replace it with a feature name.
- If they have nothing to say here, record blank and move on — do not press.

Then map it, honestly, out loud:

- For each want they named, say in one line whether the tracker does it and where ("That one it does — that's the morning brief"), or that it does it partly, or that it cannot.
- **Out of scope is said out loud, at onboarding, in plain words.** In this version the tracker does **not**: hold meal-by-meal food detail inside the tracker (your food source keeps the meals; the tracker holds day totals); sync sleep automatically from a health platform; set per-muscle set targets; or replace your calendar, email or food app. If a want lands on one of these, say: "That one it can't do in this version — I won't promise it. What it does instead is [the nearest true thing, or 'nothing — that stays with you']."
- Never promise a want the product does not have, and never quietly reinterpret a want into a different feature they did not ask for.
- Record a short mapping summary next to their words (their words stay untouched; the mapping is the agent's note, labelled as such), and read both back in Block 9.

Then say:

> "Good — that's what this setup is for. Now your goals themselves."

---

## Block 3 — Goals

### 3a. The goal in their own words

Ask:

1. "In your own words, what do you want to achieve with your training and food?"

Record their answer word for word. Do not shorten it, do not rewrite it, and do not replace it with a label. Their own words stay on the record even after a goal mode is picked below.

### 3b. Map it to one goal mode

Say:

> "The tracker works with four goal modes. Each mode is a simple idea about food energy. Your body needs a certain amount of food each day just to stay the same weight — that amount is called your maintenance. Each mode eats a set share of that maintenance amount. I will explain all four, then you pick the one that fits what you just told me. If none of them fit, we leave the mode blank and use your own words only."

Explain the four modes, in plain language, exactly like this:

- **Lose Fat** — "You eat below maintenance, at about 85% of it. Because the food does not cover the full day, your body uses some stored fat to make up the difference. Protein stays high in this mode, because protein helps protect your muscle while the fat comes off. The aim is steady fat loss, not fast loss."
- **Recomposition** — "You eat close to maintenance, at about 97% of it. You build muscle and lose fat at the same time, slowly. Your weight on the scale may move very little, but your shape and strength can still change. This is the slow, balanced mode."
- **Gain Muscle** — "You eat a little above maintenance, at about 107% of it. The small extra gives your body spare energy to repair training and build new muscle. The aim is slow gain, so most of the gain is muscle, not fat."
- **Gain Weight** — "You eat above maintenance, at about 112% of it. This is for someone who finds it hard to gain weight, or who needs to gain weight overall. The extra is larger than in Gain Muscle, so the scale should move up more clearly."

Then ask:

2. "Which of those four fits your goal best — Lose Fat, Recomposition, Gain Muscle, or Gain Weight? You can also say none, and we leave it blank."

Record:

- Goal mode: the one they pick, or blank.
- Never pick the mode for them. If their own words and the mode they pick seem different, read both back once and ask: "Just to check — you said [their words], and you picked [mode]. Keep both as they are?" Record what they confirm.

Say this, word for word:

> "Your goal mode sets the tracker's starting defaults — your calorie target idea, your macro split starting point, and what your daily recommendations push towards. It does not lock anything in. You can change your goal mode, your macro preset, and your sliders at any time, just by telling me. Changing them changes the defaults from that day on. It never changes anything you have already logged."

### 3c. Target weight (optional)

Ask:

3. "Do you have a target weight you are aiming for? You can give it in your unit, or skip this — it is optional."

Record:

- Target weight: number and unit, exactly as given, or **blank**.
- Never suggest a target weight. Never work one out for them. If they ask what their target weight should be, say: "I can't set that for you — it is your call, and if it is about health, a qualified professional is the right person to ask. We can leave it blank and the tracker still works."

### 3d. Pace preference

Ask:

4. "How fast do you want to move towards your goal — slow and easy, steady, or as fast as is sensible? There is no wrong answer, and you can change this later."

Record:

- Pace preference: in their words, or blank.
- Say once: "Pace changes how strong the daily recommendations push. It does not change your logged history, and faster is not always better — the tracker will tell you honestly if your logged weight is moving much faster or slower than your pace."

### 3e. Step goal (optional)

Ask:

5. "Do you have a daily step goal? If yes, how many steps? If not, we leave it blank."

Record:

- Step goal: as given, or **blank**. Never fill in a round number for them.

### 3f. What success looks like in 12 weeks

Ask:

6. "Last goal question. Picture yourself 12 weeks from now. What would success look like for you? It can be a weight, a weight you can lift, how your clothes fit, how you feel — whatever matters to you."

Record:

- 12-week success description: word for word, or blank.
- Say: "I will keep that saved with your goals. Your progress checks will be measured against what you just said, not against anyone else's goal."

Then say:

> "Goals are set. Next, we link your food targets to those goals."

---

## Block 4 — Goal-linked macros

### 4a. Opt in or out

Ask:

1. "Do you want to track macros — calories, protein, carbs and fat — in this same tracker, next to your training? Yes or no."
   - If no, say: "Done — macros stay off and blank, and we skip the rest of this block." Go to Block 5.
   - If yes, continue.

### 4b. The two ways a target can exist

Say:

> "There are only two ways a target gets into this tracker. One: you give me your own numbers, and I record them exactly. Two: you have no numbers, and you ask me to work them out from your goal mode — then I show you the working, label them 'calculated', and save them only if you say yes. There is no third way. I never fill in a typical target."

Ask:

2. "Do you already have daily targets? If yes, give me any of these you know: calories, protein in grams, carbs in grams, fat in grams, water. Anything you don't give me stays blank."

- If they give any targets, record them exactly as given, labelled **user-given**, and skip to 4e. Their numbers always win over any calculation.
- If they give none, go to 4c.

### 4c. Offer the goal-mode calculation

Ask:

3. "Want me to work out a starting target from your goal mode? I will show you every number I use before anything is saved. Yes or no."
   - If no, say: "No problem — all targets stay blank. You can give me numbers, or ask for the calculation, at any time later." Go to Block 5.
   - If yes, continue to 4d. **Do not calculate yet.**

### 4d. The calculation, explained before it runs

Say:

> "Here is how the calculation works, in plain language. First, the tracker estimates your maintenance — the food energy you need to stay the same weight — from your body weight and how active you are, including your daily steps if you track them. Second, it applies your goal mode: Lose Fat uses 85% of maintenance, Recomposition 97%, Gain Muscle 107%, and Gain Weight 112%. That gives a daily calorie target. Third, it splits those calories into protein, fat and carbs using a preset and two sliders, which I will explain now."

Explain the four presets, exactly like this:

- **Balanced** — "The middle option. Protein and fat are both set at a moderate level, and the rest of the calories go to carbs."
- **High Protein** — "More of the day goes to protein. This suits people who want to protect or build muscle, or who stay full longer on protein."
- **High Carb** — "Less of the day goes to fat, so more is left for carbs. This suits people who train hard and often, because carbs are the main fuel for hard training."
- **Low Carb** — "More of the day goes to fat, so less is left for carbs. This suits people who simply prefer eating that way."

Then say the preset numbers, exactly like this:

> "Each preset is just two numbers, measured per kilogram of your target weight — or per kilogram of your current weight if you did not set a target weight. Balanced: protein 2.1 grams, fat 0.95 grams. High Protein: protein 2.8 grams, fat 0.85 grams. High Carb: protein 1.9 grams, fat 0.7 grams. Low Carb: protein 2.2 grams, fat 1.4 grams."

Explain the sliders, exactly like this:

> "The preset is a starting point, not a lock. There are two sliders you can move at any time: a protein slider and a fat slider, both in grams per kilogram. Carbs have no slider, because carbs are always what is left over. Here is the maths, in plain words: protein and carbs each give about 4 calories per gram, and fat gives about 9 calories per gram. The tracker works out the calories used by your protein and your fat, takes them away from your calorie target, and every calorie left goes to carbs. If protein and fat ever use up the whole target, carbs stop at zero — they never go below zero."

Ask:

4. "Which preset do you want to start with — Balanced, High Protein, High Carb, or Low Carb? Balanced is the middle option if you are not sure, but the choice is yours. You can change it any time."

Then:

5. "Do you want to move either slider away from the preset, or keep the preset numbers as they are?"

Rules for running the calculation:

- Run it only now, only because the user said yes in 4c, and only if every input it needs exists: a current body weight given by the user, a step picture (your logged steps averaged over the last 14 days, with at least 3 logged days in that window — if you have fewer than that, the calculation cannot run yet), and a goal mode picked in Block 3.
- If any needed input is missing or blank, **do not calculate**. Say which input is missing, leave all targets blank, and go to Block 5. Never estimate a missing input to make the calculation work.
- Show the result before saving anything: maintenance estimate, goal mode and its percentage, calorie target, preset, protein and fat slider values, and the protein, fat and carb grams that come out.
- Say: "These are calculated targets, not targets you gave me. They will be saved labelled 'calculated', so you can always tell them apart from your own numbers."
- Ask: "Save these calculated targets, change the preset or a slider and work it out again, or leave targets blank?"
- Save only the version the user says yes to. If they change a slider, re-show the new result and ask again before saving.
- A calculated target never overwrites a user-given target, in this block or ever. If the user later gives their own number, their number replaces the calculated one and the change is recorded.

### 4e. Water (optional)

Ask:

6. "Do you have a daily water target you want recorded? If yes, how much? If not, it stays blank."

Record: as given, or blank. No calculation is ever offered for water.

Then say:

> "Food targets are done. Next, your gym and equipment."

---

## Block 5 — Equipment

Ask:

1. "Does your gym have a Smith machine?"
   - If yes, ask: "Do you know the bar's starting weight on that Smith machine? If you don't know, we leave it blank — every gym's is different."
   - If no or unsure: Smith bar weight = **blank**. Do not offer a number from anywhere else, and do not use any other gym's or user's bar weight.
2. "Any machines you use a lot that I should know by name? This is optional."

Record:

- Smith machine: yes / no / unsure, as given.
- Smith bar weight: exactly as given with unit, or "blank — not known".
- Machines used often: as given, or blank.

Then say:

> "Next, the long part — your current weights, exercise by exercise. We go slowly, group by group."

---

## Block 6 — Baselines, exercise by exercise

Say this first, word for word:

> "Now your current weights, exercise by exercise. Three possible answers each time: the weight, 'I do it but don't know the weight' — that goes in blank — or 'I don't do that one' — we skip it. And if you're not sure which machine I mean, I'll show you photos — tell me which one it is."

The three-answer rule, for every exercise:

- **Knows it** → record the weight exactly as given, with unit, plus sets, reps and any note.
- **Does it but does not know the weight** → record the exercise, weight = **blank**.
- **Does not do it** → skip it. Do not add it to their baselines.

Per exercise, capture: does it (yes / no), weight + unit, sets, reps, any note.

Go group by group, in this order:

- **Push:** Machine Chest Press, Incline Dumbbell Chest Press, Flat Dumbbell Bench Press, Smith Machine Bench Press, Shoulder Press, Lateral Raise, Tricep Rope Pushdown
- **Pull:** Lat Pulldown, Seated / Low Row, T-Bar Row, Machine Row, Pull-up — and for Pull-up ask: "Assisted or unassisted?"
- **Biceps:** Seated Dumbbell Curl, Barbell Curl, Hammer Curl, Machine Curl, Cable Curl
- **Lower:** Squat (Smith, barbell, or machine? — photo check), Leg Press vs Hack Squat (photo check), Leg Extension, Leg Curl, Hip Adductor vs Abductor (photo check), Romanian Deadlift / Deadlift, Calf Raise, Hip Thrust
- **Core:** "Which bodyweight core work do you do, if any?"
- **Cardio:** bike, treadmill, assault bike, walks — distance or time only if they know it.

### Photo rule — hard stops inside this block

Never guess a machine. If the user is not sure which machine or movement is meant, or the agent is not 100% sure, **stop, show photos of both options, labelled "✓ This one" / "✗ Not this one", and ask: "Which one is it?"** Log nothing specific until the user picks.

Always trigger the photo check on these look-alike pairs:

- Smith machine bench press vs free barbell bench press
- Smith machine squat vs barbell back squat
- Smith machine shoulder press vs machine (stack) shoulder press vs dumbbell shoulder press
- Hack squat vs leg press
- Chest-supported T-Bar row vs seated cable row vs Smith row vs machine row
- Hip adductor (pads push in) vs hip abductor (pads push out)
- Leg extension vs leg curl
- Close-grip Smith bench (triceps) vs normal Smith bench (pectorals)

Also trigger it on vague words, for example: "the press", "a row machine", "the leg machine", "chest thing", "some Smith thing".

Other stop conditions:

- User gives reps as a rough range → record exactly their words in the note, and set the reps field to what they confirm. Do not tidy a rough answer into a false precise number.
- User names an exercise that is not in the list → add it. The list is a starting set, never a cap. If its machine is unclear, the photo rule applies first.
- If no confident photo exists for a movement, show no photo rather than a wrong one, and ask the user to describe the machine or send a photo of it.

Then say:

> "Baselines are done. Next, how you will tell me about your food day to day."

---

## Block 7 — Food reporting style and estimates permission

Ask:

1. "How will you tell me your food, day to day? You can pick one main way: exact numbers you already know, totals from another app that you read out to me, or meal descriptions — you tell me what you ate, in words or photos."
   - If macros were turned off in Block 4, still ask this once, briefly, and record the answer — the user may turn macros on later. If they say food reporting does not apply to them, record blank and move on.

If they pick meal descriptions, ask:

2. "I can estimate the calories and macros from a meal description, but only if you want estimates. Every estimate will always be labelled 'estimate', and an estimate never overwrites a real number you give me later. Do you want estimates — yes, or numbers-only?"
   - Record the permission exactly: estimates allowed (labelled), or numbers-only.
   - If they pick exact numbers or app totals, record: numbers-only, unless they say otherwise.

Say:

> "One more rule, so there are no surprises: I never backfill a food day you did not report, and a day you only partly report is marked partial — it is not averaged as if it were a full day."

Then say:

> "That is your food reporting set. Next, a short tour of what your tracker can do, so you know everything you have."

---

## Stage D — Capability tour: what this tracker can do

Run this after Block 7 and before the Block 9 read-back. Short and plain — this is so the user knows what they have, not a sales pitch. Say:

> "Before I read everything back, here is what your tracker can do, now that it's set up around you:"

Then go through these, one line each, in this order:

- **Today page.** "One page each day: three short points on where you stand, your numbers against your targets, your last 7 days of food and steps, your weight trend, a week-on-week comparison — and one recommendation for today, based on your goal and your own data."
- **Body heat map and recommended workouts.** "After each workout, your heat map updates itself: sore, recovering, ready. Your recommended workout is built only from ready muscles, with your own weights — never a guessed weight, and never on a sore muscle."
- **Photos for machines.** "If you name a machine and I'm not sure which one it is, I show you photos and ask — I never guess a machine into your log."
- **Sync that never duplicates.** "Bookings and food totals that arrive from the things you connected land as one record each. Run a sync twice and nothing doubles. A moved booking moves the one record; a cancelled one is cancelled, and it stays cancelled."
- **Days that never close themselves.** "A food day is complete only when breakfast, lunch and dinner are all logged, or you say the day is done. A partly logged day stays open, and your averages are worked out over complete days only — with the count shown, so you can see what an average is based on."
- **The Ask Queue.** "If you turned it on: one short evening message asking only what's still missing — a meal, by name, asked gently; a session you haven't confirmed; a weight you didn't know. Nothing to ask, no message."
- **The morning brief.** "If you turned it on: one short morning message — anything still missing from yesterday, your key numbers, and one recommendation for today that follows your goal, in your words."
- **Weekly reconciliation and backups.** "If you turned it on: once a week the tracker checks its own records against its backup and your food record, and reports anything missing, doubled or mismatched. It only reports — it never changes anything by itself. Backups are kept, so a restore brings records back with their history."
- **Everything stays yours to change.** "And the charter from the start still holds: theme, targets, goal mode, exercises, meal structure, every automation and its time — tell me any time and I'll change it, read the change back, and save it only when you say yes."

Then say:

> "Now I'll read your whole setup back to you — your words where they're your words — and nothing is saved until you say yes."

---

## Block 9 — Read-back and save

Read back the whole setup, in this order, using the user's actual answers. For every field that was skipped, say the word "blank" out loud — never skip over a blank in silence.

Read back, exactly in this shape:

- **Basics:** "Name: [name, or 'blank']. Timezone: [timezone]. Weights in: [kg / lb]."
- **Training:** "Training: [types], [frequency] times a week, split: [split, or 'no fixed split']. Injury, pain or movements avoided: [as stated, or 'none stated']. Sets and reps scheme: [scheme, or 'no scheme set']."
- **Goals:** "Your goal, in your words: [their words, read back word for word]. Goal mode: [mode, or 'blank — your own words only']. Target weight: [weight and unit, or 'blank']. Pace: [pace, or 'blank']. Daily step goal: [steps, or 'blank']. Success in 12 weeks, in your words: [their words, or 'blank']."
- **What you want from the tracker:** "[their Stage C answer, read back word for word, or 'blank']" — and what I told you it can and cannot do about each part: [the Stage C mapping summary].
- **Macros:** "Macros: [on / off]." Then one of:
  - If user-given targets: "Targets you gave me: [list each set field with its value]. Targets blank: [list each blank field]."
  - If calculated targets were accepted in Block 4: "Targets calculated from your goal mode, labelled 'calculated': [calories, protein, carbs, fat]. Preset: [preset]. Protein slider: [value] grams per kilogram. Fat slider: [value] grams per kilogram. Carbs are the remainder."
  - If neither: "All targets blank."
  - "Water target: [value, or 'blank']."
- **Equipment:** "Smith machine: [yes / no / unsure]. Smith bar weight: [weight and unit, or 'blank — not known']. Machines you use often: [list, or 'blank']."
- **Baselines:** "Baselines: [X] exercises recorded with weights, [Y] recorded with weight blank, [Z] skipped."
- **Food reporting:** "Food reporting: [exact numbers / app totals read out / meal descriptions]. Estimates: [allowed, always labelled / numbers-only]."
- **Automation:** Read each one separately: "Ask Queue: [on at (time) / off]. Morning brief: [on at (time) / off]. Calendar booking sync: [on / off]. Email booking sync: [on / off]. Booking sync check interval: [every N minutes, or 'blank — booking sync off']. Steps source: [Health Connect / Apple Health / manual / none]. Macro refresh: [on at (time), reading (food tracker / manual) / off]. Steps sync: [on at (time) / off / does not apply with your source]. Weekly reconciliation: [on on (day) at (time) / off]."
- **Connectors:** "Food source: [name + key, or 'manual', or 'none']. Steps: [platform + key, or 'manual', or 'none']. Calendar: [connected / not connected]. Email: [connected / not connected]."
- Remind them once: "Your goal mode, preset and sliders can be changed at any time. Your automation choices can be changed or switched off at any time."

Read it back like it belongs to them: use their name, and read their goal and their wants back in their exact words — not a summary of them.

Then ask, with this exact wording:

> "Ready for me to save this? Reply yes to save, or tell me what to change."

Save rules:

- **Save only on an unambiguous yes to that question.**
- A "yes" that answers any other question is never consent to save.
- If the user asks for any change, make the change, then read back **only the changed parts**, and ask the exact question again. Do not save until they say yes to the new read-back.
- If the user does not answer, save nothing.

After they say yes and the save is done, say:

> "Saved. Your heat map starts empty: every muscle shows Ready, because nothing is logged yet — Ready here means 'no data yet', not 'recovered'. Your plans, sessions, goals and macros will live in your tracker Space, in the Today, Train, Calendar, Macros and More sections."

Then explain how to log, word for word:

> "After a workout, just tell me what you did. I'll ask about any machine I'm not sure of and show you photos — I won't guess, and I won't log anything until you confirm it; if you don't reply, I'll ask again in the evening Ask Queue rather than guessing. When you eat, tell me in the way you picked in setup, and I will log only what you give me. A food day counts as complete only when breakfast, lunch and dinner are all logged, or you tell me the day is done. If a day is only partly logged, it stays open — I never close a day on my own — and I'll ask you gently about the missing part, like 'What did you have for lunch?', in the Ask Queue or the morning brief, if you turned those on."

And close with:

> "And remember: your goal mode drives your recommendations by default, but you are in charge of it. Tell me any time you want to change your goal, your preset, your sliders, your targets, or any automation choice, and I will read the change back before I save it."

---

## End of onboarding — agent checklist (do not read aloud)

Before marking onboarding complete, confirm:

- [ ] Every block was asked in order, one block at a time.
- [ ] Every skipped field is stored blank — none was filled with a typical value, another user's value, or an unlabelled suggestion.
- [ ] The goal is stored in the user's own words, and the goal mode is the one the user picked, or blank.
- [ ] No target weight, step goal, or pace was suggested as fact.
- [ ] Macro targets are labelled correctly: **user-given**, **calculated** (accepted in Block 4, inputs shown), or **blank**.
- [ ] Carbs, where calculated, are the remainder after protein and fat, and never below zero.
- [ ] Smith bar weight is as given, or "blank — not known".
- [ ] Baselines follow the three-answer rule, and every look-alike machine was resolved by the photo rule, not a guess.
- [ ] Estimates permission is recorded exactly as given.
- [ ] Each automation in Block 8 has its own separate answer — all seven consent items (Ask Queue, morning brief, calendar booking, email booking, macro refresh, steps sync, reconciliation), plus the steps source pick, the food source binding, and the booking interval where booking sync is on. Nothing is on by default. Anything "on" with no time picked is recorded as do-not-send.
- [ ] The save happened only after an unambiguous yes to: "Ready for me to save this? Reply yes to save, or tell me what to change."

- [ ] The framework charter was said at the start, in full: everything customisable, every default a starting point, changeable at any time by telling the agent.
- [ ] The experience order was followed: automation first (Block 8 as Stage A), connectors second (Stage B), then basics/training, wants + goals, the remaining blocks, the capability tour (Stage D), and the read-back last.
- [ ] Connector choices are recorded per connector (food source + key, steps platform + key, calendar, email) — each optional, none assumed — and any Stage A consent a Stage B answer changed was updated on its consent row and will show in the read-back.
- [ ] What the user wants from the tracker (Stage C) is stored word for word, with the agent's capability mapping noted separately, and every out-of-scope want was said out loud as out of scope — nothing was promised that the product does not have.
- [ ] The capability tour (Stage D) was delivered before the read-back.
- [ ] It was personal: their name used naturally once known, their goals and wants read back in their exact words, questions shaped around their meals, routine and week, and no skipped answer was filled by assumption.
- [ ] If the user arrived with a filled-in `onboarding-prompt.md`, onboarding worked from it (confirming, not re-asking from zero), and its blanks were asked here — never assumed.
