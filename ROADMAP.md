# Roadmap

**Learning Atlas — where this goes next**

Last updated: August 2026

---

## Where it stands today

Two apps on a Raspberry Pi 5 with a 7-inch touchscreen.

- **Flag Game** — 40 countries, tap to explore, 10-round quiz
- **Brain Quest** — 300 questions across English, Maths and her IB unit of inquiry, drawn from the actual weekly school log

Both are single HTML files with questions hard-coded inside. Nothing persists between sessions. Content updates happen by generating a new file and copying it across with `scp`.

It works. She plays it on loop — four or five times in a sitting, unprompted. That's the baseline everything below improves on.

---

## What this actually is

Watching her play surfaced something worth stating up front.

**This measures more than it teaches.** Every session is a small assessment. She answers thirty questions; the pattern of what she gets right and wrong is a picture of where her understanding stops.

That picture is the most valuable thing the device produces — and it's currently thrown away the moment the browser closes.

So the parent view moves to the front of the roadmap. Not because reporting is more important than teaching, but because **knowing what to teach comes first**, and the teaching itself happens between the two of us at the weekend, not on a screen.

Whether the device should eventually try to teach — adapting difficulty, re-explaining what she gets wrong — stays open. That decision comes at Phase 4, with real data behind it.

---

## Phase 1 — See what she knows

*The device starts remembering, and starts telling me something useful.*

**1.1 Topic tagging**
Every question tagged with a topic and subtopic. Two levels: nineteen topics for the front page, sixty-eight subtopics for drilling in.

Without this the best the device can say is "English: 7 out of 10" — which the score screen already shows and which tells me nothing. With it, the device can say "capitals are solid, common nouns are not."

This is also the foundation for everything in Phase 4, so it isn't throwaway work.

**1.2 Storage**
Results saved in the browser's own storage on the Pi. Accumulates across sessions. Crude — it lives inside Chromium and would be lost if browser data were cleared — but it costs one session instead of ten and produces real data within a week.

**1.3 Parent view**
A screen showing per-topic accuracy. Nineteen rows, worst first. Tap a topic to see the subtopic breakdown and the actual questions she got wrong, verbatim.

The test of whether this works: does it change what I do with her on a Saturday morning? If yes, it justifies building properly at Phase 3. If I glance at it twice and stop, I've learned that for one session's cost.

**1.4 Home screen**
A launcher with two big buttons. She picks between the Flag Game and Brain Quest herself.

**1.5 Auto-launch on boot**
Flip the wall switch, wait thirty seconds, the home screen appears. No laptop involved.

Right now she can only play when I'm free to SSH in. That's the binding constraint on how often the device gets used — and therefore on how much it can tell me.

**Size:** 3–4 sessions

---

## Phase 2 — Speaker and voice

*She can practise without me reading to her.*

A USB speaker, and the browser's built-in speech synthesiser reading questions aloud. Free, offline, no API key, robotic but clear.

She is six and reading is still work. If the device reads to her, she can practise alone — which changes both when it gets used and how much data it produces.

This also matters later: the question formats in Phase 5 need instructions spoken while her hands are busy.

**Hardware:** USB speaker (~₹500–1,500)
**Size:** 1–2 sessions

---

## Phase 3 — Fresh content, automatically

*The school's weekly PDF becomes questions without me writing them.*

```
PDF  ->  extract text  ->  Claude generates  ->  validate  ->  I review  ->  publish
```

**3.1 Extraction** — pull text from the Toddle PDF. The weekly log is a clean table; the unit letter is a decorative slide deck and will extract messily. Tolerable, because an AI handles the mess downstream.

**3.2 Generation** — one Claude API call with a heavily constrained prompt. The highest-risk stage in the project: a working API call takes twenty minutes, a prompt that reliably produces *good* Grade 1 questions takes iteration. Questions come back already tagged by topic and subtopic.

**3.3 Validation** — automated checks. All fields present, correct answer among the options, no duplicate options, reading level in range, topic tag from the approved list, no banned words.

**3.4 Review** — questions print to the terminal, I read all thirty, delete or regenerate anything wrong, then publish. **Non-negotiable.** Code can verify a question is well-formed; it cannot verify it is true, age-appropriate, or emotionally sensible. She trusts this device because her father built it.

**3.5 Publish** — saved as a dated JSON file. Old weeks kept, which makes revision mode nearly free later.

**Setup:** Anthropic API key, Python environment on the Pi
**Size:** 3–4 sessions

---

## Phase 4 — The service

*The Pi becomes a server, not just a machine running a browser.*

A small Python service. Chromium stops opening a file from disk and starts fetching over HTTP from localhost.

What it unlocks:

- **Storage that survives.** The Phase 1 parent view is crude — data lives inside the browser and could vanish. This moves it to real files on disk, backed up, permanent.
- **Questions live outside the app.** Adding a question stops meaning "edit the app."
- **Reachable from a phone.** Check the parent view from the kitchen. Upload the weekly PDF from anywhere in the house.
- **The review step becomes a web page** rather than a terminal.

By now Phase 1 will have shown whether the parent view is genuinely useful. If it is, this is where it gets built properly.

**Size:** 2–3 sessions

---

## Phase 5 — Adaptive difficulty

*Where the "should it teach?" question gets answered.*

By this point there's a year's worth of nothing — or several months of real data showing exactly where she struggles. That data makes this decision, rather than guesswork.

### Needed either way

**Difficulty levels.** Every question tagged L1 (recognise), L2 (apply), L3 (stretch).

**Guess detection.** She guesses, and she replays four or five times a sitting — so later loops are part memory, part luck. Three defences:

- **Repetition with variation** — the same concept, worded differently, correct three times. The surface changes so memorising doesn't help.
- **Spacing across days** — those three confirmations must land on separate days. A Saturday marathon proves nothing; knowing it again on Wednesday does.
- **Reset on wrong** — one wrong answer zeroes the count for that concept.

**Within-day freshness.** Consecutive replays prefer unseen questions.

**Levels invisible to her.** A child who replays to beat a score will grind to level up, and grinding is guessing at speed.

### The open decision

**Does the engine serve her the right difficulty, or probe to find her limit?**

Serving the right difficulty keeps her at the productive edge — too easy is boring, too hard discourages. That's a teaching goal.

Probing deliberately tests the adjacent concept after each correct answer, walking the boundary of what she knows. That produces a sharper diagnostic picture but a less comfortable experience.

They're not incompatible — a system could probe during the first round and settle into the right difficulty afterwards. Decide with the data.

**Size:** 3–4 sessions

---

## Phase 6 — Beyond multiple choice

*Test production, not just recognition.*

Multiple choice is recognition, and guessable. These aren't:

**Tap to order** — arrange `school / go / I / to` into a sentence. Maps onto her CHIPS work, nearly impossible to guess.

**Sort into buckets** — words into Person / Place / Animal / Thing. Foods into Everyday / Sometimes.

**Type the number** — a large on-screen number pad. "3 tens and 4 ones = ___".

Two reasons these matter more than they first appear: they produce far cleaner diagnostic signal because guessing largely disappears, and they exercise recall rather than recognition, which is closer to what school actually asks of her.

Audio becomes close to mandatory here — an ordering question needs its instruction spoken while her hands are busy with tiles.

**Size:** 3–4 sessions, addable one at a time

---

## Phase 7 — Later

Listed so they aren't forgotten. Not scheduled.

- **Better voices** — cloud text-to-speech instead of the browser's robotic one. Costs per character.
- **Microphone** — reading practice, or her narrating what she learned. Child speech recognition is genuinely hard; needs a specific purpose before it's built. Buy the mic, leave it on the shelf.
- **Revision mode** — quiz her on last month, not just last week. Nearly free once dated question files exist.
- **Trends over time** — is she improving on a topic, or stuck? Comes free once several months of tagged data exist.
- **Hindi and Marathi** — Devanagari on a touchscreen for a six-year-old is a separate problem.
- **Hardware** — thermal printer, LEDs, an LED world map on her wall.

---

## Deliberately not doing

Stated so scope creep has something to argue against.

- **Cloud backend.** Everything runs on the Pi. No hosting, no accounts, no sync.
- **Multiple users or devices.** One child, one Pi. Profile data sits in a folder so a sibling is possible later; nothing is built for it.
- **Badges, streaks, gamification.** She already replays voluntarily. Adding extrinsic rewards to intrinsic motivation usually damages it.
- **Anything sending her personal data off the device.** Curriculum topics go to the API. Her name, her school, and her records stay on the Pi.
- **Showing her the diagnostics.** The parent view is for me. She sees her round score, which she already enjoys. A six-year-old does not need a dashboard of her own weaknesses.

---

## Safety, throughout

An AI generating content for a six-year-old needs more than good intentions.

**In the prompt** — age six explicitly. No violence, horror, death, anything sexual, body-image content, religion, politics, or comparison to other children. Positive framing only.

**In validation** — banned-word list, reading-level check, answer-must-be-in-options, topic tag from the approved list, flag any question whose explanation contradicts its answer.

**In review** — every question read by me before it publishes.

**In architecture** — her name and school never leave the Pi. Skill data local and gitignored. No telemetry.

Three layers, none trusted alone.

---

## Sequencing

Phase 1 first — it's what makes everything after it measurable.

Phase 2 is independent and could slot anywhere.

Phase 3 is independent of 1 and 2, but its output is much more useful once topic tagging exists.

Phase 4 gates 5 and 6.

Phase 5 wants several months of Phase 1 data behind it.

Estimates are in sessions, not weeks — roughly two hours a weekend alongside a full-time job and a six-year-old. What matters more than schedule is that every phase leaves the device working and produces something she notices.

---

## Open questions

Things that need watching before they can be answered:

- **Does she read the questions herself, or ask me to read them?** Determines how urgent Phase 2 is.
- **Where does she lose interest?** Round two? Round three? Mid-round?
- **Does she replay to learn, or to beat her score?** Affects the Phase 5 design.
- **Does the parent view change what I do with her?** The test of whether Phase 1 was worth building. Answerable within a fortnight.
