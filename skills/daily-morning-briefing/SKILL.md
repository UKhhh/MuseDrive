---
name: daily-morning-briefing
description: Build a daily morning briefing that reads like a good friend talking, not a report. Covers weather, vehicle status, health, and personalized reminders, plus one urgent news item only when one truly matters. Designed to be customized to the user's own city, car, health source, and thresholds, and to be shared with others.
---

# Daily Morning Briefing / 每日早报

A daily morning briefing skill. The goal is a short, warm, first-person
message that feels like a friend checking in — not a formal report, not a
checklist, and not a dump of everything the assistant thought about.

This skill is generic on purpose. Nothing in it is tied to one person:
fill in the Customization Worksheet at the end before first use, and
change any threshold, place, or source to fit your own situation.

## 1. One job, one message

- Use a single scheduled job for the whole briefing. Do not create
  separate jobs for weather, vehicle, health, or news.
- The job runs once every morning at a time the user chooses
  (example: 07:30 local time). It sends one message to the user's main
  chat.
- The briefing is never skipped. If one part cannot be read, send the
  other parts and say plainly, in one line at the end, which part was
  not available. Never invent a value to fill a gap.

## 2. What the briefing contains

1. **Weather** — today's conditions for the user's place, high/low
   temperature, precipitation, and wind, plus one concrete clothing
   suggestion (umbrella, extra layer, sun protection — not just
   "dress warmly"). Always add the assistant's own subjective feel
   tied to the real temperature (cold on a chilly morning, warm on a
   hot day), with a matching mood emoji — never just list numbers.
2. **Vehicle status (read-only)** — battery level and range, charging
   state and limit, tire pressures against the recommended value,
   lock state, and inside/outside temperature. Never send a vehicle
   command in this job: no unlock, no start, no charging change.
   Waking a sleeping vehicle to read its status is allowed only
   when the user has explicitly authorized it for this briefing,
   at most once per briefing, and read-only. Never retry a failed
   wake; if the vehicle still cannot be read, mark its status
   temporarily unreadable, never guess, and never disturb the
   vehicle's sleep repeatedly for briefing data. Report the
   odometer only from a live reading each run; never reuse a
   static historical figure as the current mileage.
3. **Health** — last night's sleep (duration, bed/wake times) and
   yesterday's activity (steps, resting heart rate), only if readable
   from the user's authorized health source. If sleep has not synced
   yet, re-fetch every 5 minutes until 30 minutes after the
   briefing start time. If any attempt succeeds, send the complete
   briefing immediately; do not wait for the window to end. If
   still unavailable at the end of the window, send the briefing
   without sleep data. After the briefing is sent, at most one
   sleep-only supplement is allowed if sleep syncs later — never
   resend the whole briefing. Never invent sleep data.
4. **Personalized reminders** — woven into the chat naturally (see
   section 4). Never a separate "suggestions" chapter.
5. **Urgent news (only if truly urgent)** — at most one item, only
   when it directly affects the user: a severe weather or disaster
   warning for their city, a major national emergency, a major
   vehicle recall or safety notice, or a major device security
   update. State what it is, why it matters to them, whether action
   is needed, plus source and time. If there is no such item, say
   nothing about news at all. Never pad the briefing with ordinary
   tech, entertainment, or product news, and never restate the
   weather forecast as news. Prefer official sources: the
   manufacturer's official notice or the regulator for recalls
   and safety issues, and local official meteorological or
   emergency channels for disaster warnings. News must have a
   verifiable publish or update time and must apply to the user's
   region, device, or vehicle model. Rumours, unsourced reposts,
   and old news never qualify as urgent news.

## 3. Location

Determine one place per day, in this order:

1. The user's current location, if the device shares it and it can be
   reverse-geocoded to a city or district.
2. Otherwise, a weekday/weekend fallback the user chooses, for
   example: work city on weekdays, home city on weekends.

Report the place naturally, for example "[your city] is cloudy today".
Never explain which location rule was used, and never write
"reported by current location" or "reported by the weekday rule".
Explaining the rule makes the message feel stiff and soulless.

## 4. Tone, format, and the two hard rules

**Tone:** first person, like a good friend talking. Warm and
affectionate.

**Openings:** the first paragraph is a single greeting line on its
own, with a fitting emoji matched to that day's weather, sleep, and
mood — for example "Good morning!" on a good day, gentler on a
rainy or short-sleep day. The second paragraph starts with the
actual date and weekday for that day, in the user's local time —
computed for that day, never copied from yesterday, and the weekday
must match the date. The first and second paragraphs are separated
by exactly one blank line — no extra gap, and not merged into one
paragraph. Rotate 4–5 or more friend-like greetings and emoji
combinations; never use the same greeting two days in a row, and
never repeat one greeting every day, or it gets boring. Do not use
stiff stock greetings.

**Emojis:** fitting ones, matched to weather, temperature,
wind, clothing, battery, tires, charging, and sleep. Do not pile
them up, and do not put one on every sentence. Emojis are required,
not optional: a briefing with no emojis at all is a format failure.
Every paragraph must contain at least one fitting emoji — a
paragraph with no emoji is a format failure, even if other
paragraphs have emojis. Every important point needing attention
must carry its own emoji right beside that reminder — umbrella,
clothing, wind, heat/cold, low battery, charging, tire pressure,
lock, pre-heat/pre-cool, seat heating/ventilation, sleep, poor
sleep, don't rush driving, low steps, fog, urgent news. One emoji
per important point, not a pile of three or four in a row, and
never directly after a number. When reminding the user to bring
something important, such as an umbrella on a rainy day, pair it
with its emoji, never a bare text reminder.

**Numbers:** use Arabic numerals for every measurable value —
temperature, battery percentage, range in km, tire pressure in bar,
sleep clock times and duration, sleep score, steps, resting heart
rate. Never turn a known number into a word approximation (for
example, never write "about four hundred kilometres" when the
figure is 411 km, and never write "about five and a half hours"
when the figure is 5.6 hours or 00:27–06:28). Where a number exists,
use it instead of a vague word. Write ranges in full, with a unit
on each side (for example "12°C to 14°C"), never with "~".
Write clock times as HH:MM (for example 23:49, 07:08). Tire
pressures are rounded to one decimal place, and when all four
tires meet the recommended pressure, just say all four tire
pressures are normal and do not list each tire one by one. List
individual pressures only for tires that are abnormal, still
rounded to one decimal place.

**Terminology:** the sleep quality score is always called the
sleep score, never a device score.

**Sleep score, honest emotion:** the sleep score must always carry
an emotional expression, as a good friend who dares to tell the
truth. If the score is low and the cause is late sleeping, say so
frankly and criticize the late sleeping directly — do not soften
it into nothing. If the score is high, encourage and praise it
clearly. The score is still a device reference, not a medical
judgment, but the tone must not be neutral or evasive either way.

**Hard rule 1 — no report format.** Never use bold section headings
(for example "Weather", "Vehicle", "Health", "My suggestions").
Never count suggestions ("my two suggestions are...") and never
write "suggestions are as follows". Reminders must be woven into
the chat, for example "By the way, top up the tires before you
leave" or "And since you slept little last night, don't rush
driving today".

**Hard rule 2 — conclusions only, no thinking process.** The
briefing must contain only information the user needs to know and
its conclusions. Never write out the assistant's internal thinking,
reasoning, background checklist, or judgment logic — for example,
never write "I put the weather, car, and health together and
thought about it" or "based on my maintenance background, I
judged...". Thinking is the assistant's own business; the user
only needs the result.

## 5. Dynamic, not templated

Each day's briefing must be adjusted in real time and must not read
like the same template with a new date. Before writing, combine
that day's information:

- Today's weather vs yesterday (temperature drop, rain, wind).
- Vehicle changes vs yesterday (battery drop, tire change, inside
  temperature, charging state).
- Health changes (last night's sleep, yesterday's steps).
- Weekday and location routine (workday city, weekend city, a
  Friday trip home, a Monday trip back).

Then highlight only the 1–2 most relevant points that day. Mention
normal items in one line. Emphasize anything abnormal or changed.
Vary the wording every day; never repeat the same sentence for
days on end.

**Compare only when credible — A/B/C grading.** A comparison is
worth making only when the data is credible and comparable. Grade
every comparison before using it:

- **A — complete:** both days' values are present, from the same
  source and the same time window, with clear timestamps. A
  numeric comparison is fine.
- **B — partial:** some fields are missing, or the source or time
  window does not fully match. Describe the change qualitatively
  only, and state the limitation.
- **C — missing, stale, or abnormal:** no comparison, no guessing.

Distinguish yesterday's observed weather from yesterday's forecast;
never present a forecast as observed, and say so explicitly when
only a forecast is available. Compare activity data only for the
same statistical day, and vehicle data only with the time gap
between the two readings in mind. If yesterday's record is missing
or incomparable, omit the comparison and note why in the run
record. Never force a comparison to fill the format.

**Data reliability — check source and time.** Record the source
and reading time of every key value where possible, and prefer
data read live in this run. If a cached value is used, check its
age; a stale cache must never be presented as live status. Mark
each module's data as read successfully, failed, or uncertain.
Re-read or double-check any clearly abnormal value, missing field,
wrong unit, implausible timestamp, or value far from recent
records; if it is still abnormal, mark it "unconfirmed" and do
not report it as fact or base a strong reminder on it. Never fill
a gap with an estimate. Always use the latest synced values when
a health source re-syncs and revises a session.

**Daily freshness — find one real observation.** The briefing is
not yesterday's data with a new date: each day, find one thing
from the real data that is worth the user's attention, and give
the right information or action for it. Freshness comes from real
change, trends, links, and useful reminders — never from
rewording, inventing a change, or forcing a topic every day.

Before writing, work through these steps internally; do not show
the analysis itself to the user:

1. Check data completeness: source, collection time, update time,
   and trust level of each module. Missing or stale data must not
   be dressed up as current.
2. Find real changes: compare with yesterday, the last 7 days, or
   the last valid record, and identify changes that have a basis.
   Compare only when the data uses the same units and comparable
   time ranges.
3. Judge whether today is worth flagging: distinguish needing
   action, worth attention, normal fluctuation, and fully stable.
4. Pick the day's observation angle from the list below; at most
   one auxiliary angle may be added.
5. Form an actionable suggestion: if there is real action value,
   say clearly what to do and when; if not, say briefly that
   everything is normal and do not force a suggestion.
6. Check repetition against the last 7 days of briefing records,
   and avoid repeating the same observation, reminder, opening,
   or closing.

Observation angles — pick the most valuable one each day; do not
rotate through them like a checklist, and do not require a
different angle every day:

- **Change day:** something changed noticeably vs yesterday.
  Explain the size of the change and what it means in practice.
- **Action day:** something needs the user to act or prepare
  today. The reminder must have a clear basis.
- **Progress day:** a data-supported positive development over
  recent days. Acknowledge it moderately; do not exaggerate.
- **Reassurance day:** key indicators are stable and nothing
  needs handling. Say plainly that everything is normal, and keep
  the briefing short.
- **Trend day:** a single day looks unremarkable, but the last 7
  days show a trend worth noting. Explain the trend and its limits.
- **Planning day:** combine weather, vehicle state, and the user's
  confirmed plans to suggest today's preparation. Without reliable
  schedule information, never guess the user's plans for the day.

At most **1 main observation point + 1 auxiliary observation
point** per day; do not expand every module. Priority order:
safety risks, emergencies, or abnormalities that need timely
handling first; then time-sensitive actions or reminders; then
changes with reliable data and real meaning for the user; then
positive developments or continuing trends worth acknowledging;
then a stable state. Keep the fixed module order from section 2
(weather, vehicle, health, reminders, urgent news). Freshness
comes from what is selected inside the modules, how deeply it is
explained, and links between modules — never from reordering the
modules.

**Cross-module links — only with evidence.** When two or more
modules have a real, explainable connection, one sentence may tie
them together (for example, a temperature drop plus tire pressure
or range; sleep plus activity suggesting rest or exercise). Every
link must be able to point at its data. Without a link, report
each module on its own; never invent a causal story for the sake
of narrative.

**7-day freshness record.** After each successful briefing, save a
short record for the next 7 days of de-duplication and continuity:
date; the main observation angle used; the main observation point
and the modules it touched; the concrete suggestion or reminder
given; whether anything is still unresolved; the opening and
closing wording used; and what repeated from earlier days and
whether the repetition was still necessary. Rules: if the same
matter has not changed for two days in a row, stop expanding it
from the third day and keep at most one short status line if it
is still unresolved; remove finished, resolved, or expired
reminders from later briefings; even an important repeated item
must say why it still needs repeating today (for example, a
deadline is near or the risk is not gone); avoid repeating the
same opening, closing, or metaphor within 7 days, but prefer
natural wording over mechanically chasing variety; never rotate
angles just to fill all seven — real data and usefulness always
come first.

**Quiet days are allowed.** If nothing changed noticeably, there
is no urgent news, and nothing needs the user to act, keep the
briefing short. Say in a light, natural way that everything is
steady. Never invent a weather change, a health trend, a vehicle
abnormality, or a user schedule; never inflate a normal fluctuation
into a risk; never change how a number is expressed or hide
important information just to look different; never fill space
with empty encouragement, baseless praise, or generic advice;
and never read "different every day" as "a new discovery every
day".

**Weekly light review.** Once a week, review the last 7 briefings
for trends that are actually worth noting — sleep duration and
score trends, whether activity is rising, falling, or stable,
whether the vehicle shows a continuing state change or a pending
item, whether reminders were completed on time or repeatedly
missed, and whether the briefing itself had repeated reminders,
useless content, or data-quality problems (sleep-sync success
rate, missing or abnormal data, repeated wording, on-time sending,
duplicate sends). Present only conclusions with enough data
behind them, and state clearly where the sample is too small or
data is missing. Never use one week of data as a health judgment.

**Variation variables.** Use these to keep each briefing fresh
and interesting, without inventing anything:

- Opening greeting rotation: rotate 4–5 friend-like openings,
  never the same one two days in a row; let the tone follow the
  weather and sleep (gentler on rainy or short-sleep days, lighter
  on good days).
- Yesterday-vs-today comparisons: temperature, wind, rainfall,
  battery and range, tire pressures, sleep duration and score,
  steps — always with real numbers, and only when the comparison
  is credible (see A/B/C grading above).
- Weekday and trip routine: Monday back to the work city, Friday
  home, weekends at home; seasonal change changes the advice.
- Weather feel: rotate real humidity, wind speed, rainfall in mm,
  and day-night temperature gap; clothing combinations follow the
  actual weather, not a fixed outfit line.
- Vehicle mini-narrative: battery and mileage trends, inside
  temperature feel.
- Health narrative: vary the tone with the data (praise good
  sleep, gentle warning on short or late sleep, encouragement on
  steps); mention only the most relevant health detail that day.
- Yesterday callback: when something flagged yesterday changed
  today (tires inflated, battery charged, sleep improved), note
  the change; if nothing changed, do not force a callback.
- Closing reminder rotation: one most-fitting closing per day,
  never the same closing every day. The closing must name today's
  concrete point in one natural sentence — never a generic blessing
  stock phrase that could fit any day.
- Emoji combinations: follow that day's lead topic, a few only,
  never a fixed set.
- Anti-repetition check: before writing, glance at the previous
  day's briefing and the 7-day freshness record, and avoid
  repeating its opening, lead sentence, and closing verbatim.
- Fixed section order: keep the order from section 2 (weather,
  vehicle, health, reminders, urgent news if any). Highlight the
  1–2 most relevant points inside that order, but do not reorder
  sections to lead with the abnormal item. Reordering by
  abnormality is not used.

**Emotional value throughout the whole briefing.** Emotional
value is not limited to the moment a goal is completed. It runs
through the entire briefing: praise good sleep and good steps,
encourage gently on short sleep or few steps, acknowledge effort
and progress, and close warmly — always tied to real data, never
empty praise, never overdone, never at the expense of accuracy.
When a previously flagged item returns to normal — for example,
tire pressures back around the recommended value after days of
being low — say so warmly and give positive encouragement, not
just a dry number.

**Before sending, check:**

- [ ] Module order is still weather → vehicle → health →
  reminders → urgent news.
- [ ] Today has at least one real, valuable observation point; if
  not, a short reassurance briefing is allowed.
- [ ] At most one main observation point and one auxiliary point.
- [ ] Comparison data uses the same units, and its source and
  time are reliable (see A/B/C grading).
- [ ] Every suggestion has a clear basis, and the user can tell
  from it whether action is needed.
- [ ] No resolved matter is repeated.
- [ ] Nothing was invented for freshness — no made-up change,
  cause, or schedule.
- [ ] Sleep data still follows the retry window in section 2;
  freshness never changes that rule.

## 6. Vehicle maintenance: background thinking, trigger-only output

Maintenance knowledge is background the assistant thinks through
every day. It is **not** a list to read out every morning. Only
mention a maintenance point when its trigger is actually met, and
then only as one natural sentence with its conclusion.

Default triggers (adjust every one to the user's car and habits):

- **Battery life:** below about 20% — note it is time to think
  about charging; below about 10% — charge promptly and do not
  leave it at low charge overnight. For daily city use, a limit
  around 80% is enough; raise it before a long trip. Do not leave
  a full charge sitting unused for days, and do not push hard at
  low charge or on a long high-speed run at low charge.
- **Cabin temperature:** below about 5°C — remind the user to
  pre-heat and use seat heating before departure; above about
  35°C — remind them to pre-cool and ventilate, and not to put a
  child straight into a hot car. In cold weather, note that the
  battery itself is cold, so charging will be slower and
  regenerative braking weaker, and pre-conditioning before
  charging helps. When the temperature is comfortable, just say
  so in one line ("the car is a comfortable temperature, no need
  to fuss").
- **Tires:** a tire pressure that stays low wastes energy and wears
  the tire unevenly. On a sharp overnight temperature drop
  (about 8–10°C or more), check tire pressure once and note that
  pressure falls roughly 0.1 bar per 10°C. Name any low tire, how
  far below recommended it is, and that it should be inflated
  cold before departure; avoid high-speed long-distance driving
  until it is inflated. If pressures are normal, say so in one
  line and move on.
- **Mileage-based items:** tire rotation about every 10,000 km,
  cabin air filter about every 2 years, and similar items the
  user cares about. Mention one only when its mileage or time
  point is actually approaching. Always read the odometer live
  each run; never use a static historical figure as the current
  mileage, and only compute a mileage-based reminder from a live
  reading — if the live reading fails, say the mileage cannot be
  confirmed this time and skip the calculation.

What **not** to include every day: scheduled-charging settings,
the full battery-life explanation, the full temperature
explanation, or the full maintenance list. If the user says a
topic should not appear in the briefing at all (for example,
scheduled charging), remove it from the job and do not mention it
again unless they ask for it back.

## 7. Health: gentle, never medical

- Sleep score, if the health source provides one, is a
  device-calculated reference only — never a medical judgment.
  Mention it naturally when readable ("you slept over 7 hours
  last night, score 95, looks good"); omit it entirely when it
  has not synced even after the retry window in section 2. If the
  health source provides them, sleep efficiency, deep/core/REM
  durations, and wake count may be mentioned too.
- Only gentle lifestyle reminders: short sleep means "don't rush
  driving today, rest if you can"; good sleep means "you slept
  well, you should feel good today"; few steps means "walk a bit
  more today if you get the chance". No diagnosis, no frightening
  language.
- Do not chase weight daily. Weight belongs in a separate,
  less frequent check-in (for example, a weekly weigh-in) unless
  the user asks otherwise.

## 8. Testing before formal use, and saving resources

- **Test fully before formal use.** When the format changes, run
  manual test versions, check them, fix anything stiff or wrong,
  and retest. Only the tested version goes out as the formal
  morning briefing.
- **Wording-only changes do not re-run data collection.** If the
  user only wants a tone, wording, or structure change, edit the
  previous briefing draft in place and save it as the latest
  version. Do not re-query weather, wake the vehicle, or re-read
  health data just to test wording. Re-collect fresh data only
  when the user asks for a fresh test, or when the formal
  scheduled run fires.
- **A re-run is a fresh data pull.** When a test run is actually
  triggered, it re-queries everything, so values will differ
  slightly from the last run (battery, inside temperature, and so
  on). That is expected, not an error.

## 9. Failure handling

- Vehicle data unavailable (token expired, wake failed, network
  failed): send the rest of the briefing and add one truthful
  line at the end, for example "today's car status could not be
  checked". Never let a vehicle failure suppress the briefing.
- Wake failed: do not retry the wake. Mark the vehicle status
  temporarily unreadable and send the rest of the briefing.
- A value that fails its re-check (see section 5) is reported as
  unconfirmed, not as a fact.
- Weather or health unavailable: say plainly which part could not
  be read. Never invent a temperature, a pressure, a sleep
  duration, or a score.
- A vehicle command is never sent in this job, even if the user
  mentions one in passing. Commands need their own explicit
  confirmation in a separate conversation.

## 10. Scheduled timing: expect a few minutes' delay

The scheduled time is when the worker starts, not when the message
lands in chat. A normal run takes about 4–5 minutes from start to
delivery, occasionally around 9 minutes, because the worker pulls
weather, wakes the vehicle once (only if the user explicitly
authorized waking for this briefing) and reads its status, reads
health data, composes the briefing, and then hands it to the main
chat for delivery. This is not a fixed delay setting, and it is not
caused by the user's network: the same run is slow or fast on the
same network on different days. If the briefing must appear in
chat by an exact clock time, schedule the job about 5 minutes
earlier than that time.

## Customization Worksheet

Fill this in before first use. Everything else in this skill
works unchanged.

- Morning time (local time): __________
- Address the user as: __________
- Location priority: current device location first, then
  fallback: weekdays __________, weekends __________
- Vehicle: make/model, data source or API, recommended tire
  pressure: __________
- Health source: __________
- Battery thresholds: note charging at ______%, urgent at ______%
- Cabin temperature thresholds: pre-heat below ______°C,
  pre-cool above ______°C
- Tire rotation interval: __________ km; other mileage/time
  reminders: __________
- Topics to never include in the briefing: __________
- Urgent-news sources and the user's city/region: __________
- Weekly weigh-in or other separate check-ins: __________

## What this skill does not do

- It does not control a vehicle, change charging, or unlock anything.
- It does not diagnose health or replace a clinician.
- It does not push real-time vehicle alerts. A scheduled morning
  poll is not a real-time monitor; urgent vehicle events still
  belong to the manufacturer's own app.
- It does not wake a sleeping vehicle without the user's explicit
  authorization, and it never sends a vehicle command.
- It does not contain any user's personal data, tokens, keys,
  vehicle identifiers, or account details. Those live only in the
  user's own secure storage and in their own job configuration,
  never in this shared skill file.
