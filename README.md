# Learning Atlas

**A Raspberry Pi learning device I'm building for my six-year-old daughter — twenty years after my last commit.**

---

## What this is

A touchscreen device that sits on a desk at home. My daughter taps it. Right now it does two things:

- **Flag Game** — 40 countries. Tap a flag, learn the capital and one fact. Then a ten-round quiz.
- **Brain Quest** — 300 questions drawn from her actual Grade 1 syllabus, across English, Maths, and her IB unit of inquiry.

It runs on a Raspberry Pi 5 with a 7-inch touchscreen, in a red plastic case, next to a plant.

She played the flag game five times in a row the first evening. That's the only metric I care about so far.

---

## Why I'm doing this

Two reasons, and both are honest.

**The first is her.** I wanted a weekend ritual that wasn't a screen handing her someone else's content. Something we built, that changes when I change it, that knows what she's studying at school this week.

**The second is me.** I'm a product person. My last real commit was around 2005. Since then I've spent two decades briefing engineers, reading specs, nodding in architecture reviews, and quietly not being able to tell when I was being told the truth.

I wanted that gap closed. Not to become an engineer — to become the kind of product leader who can read the code, ask the second question, and know when an estimate is real.

Building something my kid actually uses turned out to be a much better forcing function than a course.

---

## The insight that changed the plan

I built this as a learning tool. Watching her use it, I realised it's a **diagnostic** tool.

Every session is a small assessment. She answers thirty questions, and the pattern of what she gets right and wrong is a picture of where her understanding stops. That picture is the most valuable thing the device produces — and right now it's thrown away the moment the browser closes.

So the roadmap changed. The parent view — a screen telling *me* which topics have landed and which haven't — moved from a distant phase to the very next thing built.

The teaching still happens between the two of us on a Saturday morning. The device's job is to tell me what to teach.

---

## What's actually built

| | Status |
|---|---|
| Raspberry Pi 5 assembled, OS installed, headless setup over SSH | Done |
| 7" touchscreen working, kiosk mode, touch scrolling | Done |
| Flag Game — 40 countries, quiz mode, scoring | Live |
| Brain Quest — 300 curriculum questions, 3 rounds of 10 | Live |
| Version control and this repo | Done (finally) |
| Topic tagging and parent view | Next |
| Weekly pipeline: school PDF → AI-generated questions | Designed, not built |
| Adaptive difficulty with guess detection | Designed, not built |

---

## How it's being built

This is the part I want to be straight about, because it's the actual story.

**I'm not writing this code by hand. I'm directing an AI to write it, and I'm learning by reading, debugging, testing, and deciding.**

The workflow looks like this:

1. I decide what to build and why
2. I describe it in as much detail as I can hold in my head
3. Claude writes the code
4. I read it, run it, break it, and come back with what went wrong
5. We iterate until it works on the actual hardware, with the actual kid

What I've found: **the code is the cheap part now.** The expensive parts are knowing what to build, spotting when the machine is confidently wrong, holding scope when everything sounds like a good idea, and debugging reality — which does not read documentation.

Two examples of the machine being confidently wrong, both of which I caught and corrected:

- It told me a heatsink wasn't part of my case kit. The box said otherwise, in print.
- It then told me I'd stuck that heatsink on the wrong chip and had me trying to prise it off a live board. I found a reference photo. I'd placed it correctly.

Both times I had to push back on something delivered with total confidence. That instinct — knowing when to trust the expert in the room — is the thing I actually came here to build.

---

## What it took (the unglamorous log)

Because most repos delete this part, and it's the part that would have helped me:

- Forgot the Pi password within a week. Reflashed the whole SD card.
- Lost twenty minutes to a file saved as `study.html.save` instead of `study.html`.
- A system upgrade stalled at 22%, got killed by a stray Ctrl+C, and had to be restarted with output redirected to a log so it would survive a dropped SSH session.
- The SD card adapter's write-protect switch was in the wrong position and reported the card as read-only. Ten minutes.
- Ninety minutes of network debugging one evening — to push three files.
- `.local` hostname resolution has never worked from my laptop. I use the IP address like a caveman and it's fine.

None of this appears in tutorials. All of it is what building is.

---

## Architecture

Deliberately simple. Two self-contained HTML files, each with structure, styling, data and logic in one place. Chromium runs them full-screen in kiosk mode.

```
~/atlas/
├── atlas.html      # Flag Game — 40 countries
├── study.html      # Brain Quest — 300 questions
├── README.md
├── ROADMAP.md
└── .gitignore
```

This is not how it should be built long-term. Data and code are welded together — adding a question means editing the app. Nothing persists between sessions. That's the exact pain driving the next phases.

**Where it's going:** topic-tagged questions, stored results, a parent view showing where she's struggling, then a Python service on the Pi and a weekly pipeline that turns her school's PDF into fresh questions automatically.

The current design is naive on purpose. It works, she uses it, and it taught me precisely which parts hurt. That's a better spec than anything I'd have written upfront.

---

## Hardware

| Component | Notes |
|---|---|
| Raspberry Pi 5 (8GB) | The brain |
| Official Pi 5 case with fan | Includes a heatsink. It goes on the CPU. |
| Waveshare 7" HDMI touchscreen | 1024x600, HDMI for video + USB for touch |
| 27W USB-C power supply | Use the official one |
| 128GB microSD | Reflashed more times than I'd like |
| Micro HDMI to HDMI adapter | The Pi 5 has micro HDMI. My cable didn't. |
| USB speaker | Not bought yet — Phase 2 |

---

## Running it

```bash
# Flag Game
DISPLAY=:0 chromium --kiosk --password-store=basic --touch-events=enabled \
  file:///home/sunil/atlas/atlas.html

# Brain Quest
DISPLAY=:0 chromium --kiosk --password-store=basic --touch-events=enabled \
  file:///home/sunil/atlas/study.html
```

`--touch-events=enabled` is not optional. Without it, touch drag doesn't scroll and the only way to move down the page is the scrollbar — which a six-year-old will not find.

---

## Roadmap

Seven phases, from making the device tell me what she knows through to voice and question formats beyond multiple choice.

**Phase 1** — Topic tagging, stored results, parent view, home screen, auto-launch
**Phase 2** — Speaker and text-to-speech, so she can practise without me reading
**Phase 3** — Weekly pipeline: school PDF → generated questions → review → publish
**Phase 4** — Python service on the Pi, so storage survives properly
**Phase 5** — Adaptive difficulty, with guess detection across days
**Phase 6** — Question formats beyond multiple choice
**Phase 7** — Microphone, better voices, revision mode

Full detail, including what's deliberately *not* being built and how content safety is handled: **[ROADMAP.md](ROADMAP.md)**

---

## What I've learned so far

**Hardware is unforgiving and software is forgiving.** A thermal pad is stuck once. A line of code is a Ctrl+Z. I knew this in theory. I know it now in my hands, and it's changed how I sequence work.

**Writing code is no longer the bottleneck.** Knowing what to build, holding scope, and telling when something is subtly wrong — that's the whole job now.

**Ship the naive version.** The single-file app with hard-coded data is architecturally poor and it produced the best outcome in this project: a kid playing on loop. The pain of maintaining it is now teaching me exactly what the good version needs.

**A real user beats a spec.** She found problems in ten minutes that I wouldn't have found in ten hours — that touch scrolling didn't work, that she'd replay to farm her score rather than to learn.

**Watch before you plan.** The single most useful insight in this project — that it's a diagnostic tool, not a learning tool — came from watching her play, not from designing. It reordered the entire roadmap.

---

## A note on this being public

This is a learning log as much as a project. The code is amateur in places, the architecture is deliberately simple, and the commit history will show me learning in public.

That's the point. Twenty years is a long time to go without shipping anything yourself.

---

*Built in Mumbai, on weekends, with a six-year-old as the QA team.*
