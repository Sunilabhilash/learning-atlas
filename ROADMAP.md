# Roadmap

**Learning Atlas — where this goes next**

Last updated: August 2026

---

## Where it stands today

Two apps running on a Raspberry Pi 5 with a 7-inch touchscreen.

- **Flag Game** — 40 countries, tap to explore, 10-round quiz
- **Brain Quest** — 300 questions across English, Maths and her IB unit of inquiry, drawn from the actual weekly school log

Both are single HTML files. Questions are hard-coded inside the app. Nothing persists between sessions. Content updates happen by generating a new file and copying it across with `scp`.

It works, and she plays it on loop. That's the baseline everything below improves on.

---

## The five capabilities being built

Not a sequence — five things with dependencies between them.

| Capability | What it does | Depends on |
|---|---|---|
| **Home screen** | She picks between apps herself | Nothing |
| **Audio out** | Questions read aloud | Nothing |
| **Content pipeline** | School PDF → questions, automatically | Nothing |
| **The service** | Pi serves apps over HTTP, can save state | Nothing (but everything below needs it) |
| **Adaptive engine** | Tracks what she actually understands | The service |

Two of these float free and could be built in any order. The rest queue behind the service.

---

## Phase 1 — Independence

*Goal: she can use the device without me operating it.*

**1.1 Home screen**
A launcher with two big buttons. She taps to choose between the Flag Game and Brain Quest, and taps back to return. Right now switching apps requires me to SSH in and run a command — which means she can only use it when I'm free.

**1.2 Auto-launch on boot**
Flip the wall switch, wait thirty seconds, the home screen appears. No laptop involved.

**1.3 Speaker + text-to-speech**
A USB speaker, and the browser's built-in speech synthesiser reading questions aloud. Free, offline, no API key. The voice is robotic but clear.

This is the change that matters most in this phase. She is six and reading is still work. If the device reads to her, she can practise alone — which changes when and how often it gets used.

**Hardware needed:** USB speaker (~₹500–1,500)

**Rough size:** 2–3 sessions

---

## Phase 2 — Fresh content, automatically

*Goal: the school's weekly PDF becomes questions without me writing them.*

The pipeline, in six stages:

```
PDF  →  extract text  →  Claude generates  →  validate  →  I review  →  publish
```

**2.1 Extraction** — pull text out of the Toddle PDF. The weekly log is a clean table; the unit letter is a decorative slide deck and will extract messily. That's tolerable because the next stage is an AI that handles mess.

**2.2 Generation** — one Claude API call, with a carefully constrained prompt. This is the highest-risk stage in the entire project: getting a prompt that reliably produces *good* Grade 1 questions takes iteration, not just a working API call.

**2.3 Validation** — automated checks. Every question has all its fields, the correct answer appears in the options, no duplicate options, reading level within range, no banned words.

**2.4 Review** — questions print to the terminal, I read all thirty, delete or regenerate anything that looks wrong, then publish. **This step is non-negotiable.** Code can check that a question is well-formed; it cannot check that it is true, or age-appropriate, or emotionally sensible. A six-year-old trusts this device because her father built it. Nothing reaches her unreviewed.

**2.5 Publish** — saved as a dated JSON file. Old weeks are kept, which gives revision material for free later.

At the end of this phase the app still has questions embedded and I still copy files across manually. That's deliberate — this phase proves the questions are good before anything is built on top of them.

**Setup needed:** Anthropic API key, Python environment on the Pi

**Rough size:** 3–4 sessions

---

## Phase 3 — The service

*Goal: the Pi becomes a server, not just a machine running a browser.*

A small Python service on the Pi. Chromium stops opening a file from disk and starts fetching from `localhost`.

This is a bigger change than it sounds. It unlocks:

- **Questions live outside the app.** Adding a question stops meaning "edit the app."
- **The app can save things.** Scores, skill state, session history — nothing persists today.
- **Reachable from a phone.** Upload the weekly PDF from anywhere in the house.
- **Everything in Phases 4 and 5** depends on this existing.

Also in this phase: the manual review step from 2.4 becomes a web page. Drop the PDF in, see the generated questions, edit or delete, publish.

**Rough size:** 2–3 sessions

---

## Phase 4 — Adaptive difficulty

*Goal: the questions fit what she actually knows.*

Two problems to solve, and the second is the harder one.

**4.1 Difficulty levels**
Every question tagged L1 (recognise), L2 (apply), or L3 (stretch). The engine serves questions at her current level per topic, not a single global difficulty.

**4.2 Guess detection**
She guesses. On a three-option question that's a 33% chance of a right answer meaning nothing. Worse, she replays the game four or five times in a row, which means later loops are partly memory and partly luck — and a naive system would promote her to L3 by dinner.

The defence has three parts:

- **Repetition with variation.** The same concept must be answered correctly three times, worded differently each time. She can't memorise the surface because the surface changes.
- **Spacing across days.** Those three confirmations must land on separate days. A Saturday marathon doesn't promote her; knowing it again on Wednesday does.
- **Reset on wrong.** One wrong answer on a concept zeroes its count. Harsh, but two rights followed by a wrong usually means the rights were luck.

**4.3 Within-day freshness**
Consecutive replays prefer questions she hasn't seen yet today. Otherwise the third loop is half déjà vu.

**4.4 Parent view**
A simple page: which topics she's solid on, which she's struggling with. Not a report card — a prompt for what to work on together at the weekend.

**Levels stay invisible to her.** A child who replays to beat a score will grind to level up, and grinding is guessing at speed. She sees her round score, which she already loves. The mastery ledger stays with me.

**Rough size:** 3–4 sessions

---

## Phase 5 — Beyond multiple choice

*Goal: test production, not just recognition.*

Multiple choice is recognition — and guessable. These formats aren't:

**5.1 Tap to order** — arrange `school / go / I / to` into a sentence. Maps directly onto her CHIPS sentence work, and is nearly impossible to guess.

**5.2 Sort into buckets** — drag words into Person / Place / Animal / Thing. Or foods into Everyday / Sometimes. Good for nouns and for the healthy-choices unit.

**5.3 Type the number** — a large on-screen number pad. "3 tens and 4 ones = ___". Production, not selection.

**Audio becomes close to mandatory here.** An ordering question needs its instruction spoken while she manipulates tiles; a number-pad question is easier to hear than to read while her hands are busy. This is why the speaker comes early.

Each format needs a new interaction component, new validation, and new generation prompts. Roughly a session each, and they can be added one at a time.

**Rough size:** 3–4 sessions, spread out

---

## Phase 6 — Later

Not scheduled. Listed so they don't get forgotten.

- **Better voices** — cloud text-to-speech instead of the browser's robotic one. Costs money per character; worth it if she uses audio heavily.
- **Microphone** — reading practice, or her narrating what she learned. Child speech recognition is genuinely hard and this needs a real reason before it's built. The mic can be bought and left on the shelf.
- **Revision mode** — quiz her on last month, not just last week. The dated question files make this nearly free once the service exists.
- **Hindi and Marathi** — Devanagari on a touchscreen for a six-year-old is a separate problem.
- **The hardware roadmap** — thermal printer, LEDs, an LED world map on her wall.

---

## Deliberately not doing

Stated so scope creep has something to argue against:

- **A cloud backend.** Everything runs on the Pi. No hosting, no accounts, no sync.
- **Multiple users or devices.** One child, one Pi. Profile data lives in a folder so a second child is possible later, but nothing is built for it.
- **Badges, streaks, gamification.** She already replays voluntarily. Adding extrinsic rewards to intrinsic motivation usually damages it.
- **Anything requiring her personal data to leave the house.** School curriculum topics go to the API. Her name, her school, and her learning records stay on the Pi.

---

## Safety, throughout

An AI generating content for a six-year-old needs more than good intentions.

**In the prompt:** age 6 explicitly, no violence, no horror, no death, nothing sexual, no body-image content, no religion or politics, no comparison to other children. Positive framing only.

**In validation:** banned-word list, reading-level check, answer-must-be-in-options, flag any question where the explanation contradicts the answer.

**In review:** I read every question before it publishes.

**In architecture:** her name and school never leave the Pi. Skill data stays local and gitignored. No telemetry.

Three layers, none trusted alone. Any one can fail; all three failing at once is unlikely.

---

## Sequencing

Phases 1 and 2 are independent — either could come first.

Phase 3 gates everything after it.

Phase 4 needs 3. Phase 5 wants 1.3 (audio) in place.

The honest position on timing: this is roughly two hours a weekend, alongside a full-time job and a six-year-old. Phase estimates are in sessions, not weeks, because weekends get eaten. What matters more than the schedule is that each phase leaves the device working and produces something she notices.
