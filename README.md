# Pip: Your Tiny Study Buddy

**Pip figures out *why* you're stuck, then does something about it.**

![Pip, your tiny study buddy](pip_ui_preview.png)

A shame-free study buddy for students with ADHD (and anyone who has stared at an assignment for three hours without starting it), powered by three NVIDIA Nemotron models working as one agent.

Built at **tidalBYTE '26** (Texas A&M) by Ava and Rana, two students with ADHD. Entered for **Overall** and **Best Use of NVIDIA Nemotron**.

> Most productivity apps treat every stall the same: "just focus." Psychology says there are four different reasons you're stuck. Pip uses Nemotron to work out which one it is, then picks a fix from the field that studies it, and actually sets it up for you.

---

## Inspiration

> **Our mission: help every distracted brain take back its attention, one tiny step at a time.**
>
> Hi! We're Ava and Rana, the two ADHD college students behind Pip. We care, we put in the effort, and we still get stuck trying to start the "easy" or "basic" tasks. Attention has become one of the most valuable things in the world, and so much of modern life, especially endless short-form content, is built to take it from us. We don't think we're the only ones feeling that pull. So we made Pip to pull back: Pip helps you figure out why you're stuck, take one tiny step, and put your most valuable resources (your attention, time and energy) toward the life you want, taking a little weight off your day and supporting your mental health along the way.
>
> And why a bird? Pip is a shima enaga, a tiny long-tailed tit from Hokkaido, Japan, often called a "snow fairy." They're a symbol of purity and serenity. We hope Pip brings you a little serenity, too.

We both have ADHD. "Just focus" has never once worked for us. Neither have to-do apps, because the to-do list was never the problem. The problem is the *wall* you hit before starting, and that wall isn't always the same wall:

- Sometimes the task feels too big to even begin.
- Sometimes it feels pointless.
- Sometimes your phone wins every single time.
- Sometimes the deadline just doesn't feel real yet.

Procrastination research has a name for this. **Temporal Motivation Theory** (Steel, 2007) says:

> **Motivation = (Expectancy × Value) ÷ (Impulsiveness × Delay)**

Four levers, four different jams, four different fixes. ADHD research adds that two of those levers (impulsiveness and delay, a.k.a. *time blindness*) are especially loud for ADHD brains. We wanted a buddy that figures out *which* lever is jammed for *you, right now*, and then helps, instead of nagging.

## What it does

1. **You check in, messily.** Type (or say) whatever's going on: *"lab report due Tuesday, I keep opening TikTok."* Enter sends; no forms.
2. **Pip screens for safety first.** If it sounds like real distress, Pip stops coaching and points to the 988 Suicide & Crisis Lifeline and campus counseling.
3. **Pip diagnoses the jam.** A Nemotron reasoning model scores all four levers and explains its insight in plain words, e.g. *"You can't see the next sentence yet, so the task feels like a wall instead of a step,"* plus a tiny thing to try. Pip's face changes to match (wobbly, shrug, dizzy, sleepy).
4. **Pip picks a research-backed trick** for that lever (tiny first step, if-then plan, body doubling, mini-deadline, temptation bundling…), each labeled with the field it comes from.
5. **Pip actually does it.** A fast Nemotron model calls tools that create a 10-minute first step, a calendar block you can download as `.ics`, a scheduled check-in, or a "why this matters to you" link, timed to when you focus best.
6. **Focus timer.** A Pomodoro timer lives in the top corner: 15/3, 25/5 or 50/10 minute rounds, a long break every 4th round, a chime when it's time, and the countdown in your browser tab so time stays visible.
7. **Pip checks in on the task.** A checklist of exactly what Pip set up, then: *It's going well! / Still stuck, new trick / Remind me later.*
8. **Pip learns what works for you.** Every check-in updates a per-student record of which tricks helped. Next time, Pip says things like *"'If-then plan' helped 0 out of 1 times for you, so let's try 'Body double' instead."*
9. **My week** turns syllabi into a calendar. Drop in syllabus PDFs and Nemotron extracts every graded item, due date, time and weight. You get:
   - a color-coded week view and month view
   - "Coming up" with urgency pills
   - Pip's reminders, each matched to the assignment it's about and showing a live countdown

Plus everything that makes it bearable to open when your brain is fried:
- dyslexia-friendly type, light and dark mode, gentle animated waves
- optional focus sounds (rain, birdsong, ocean, white noise, music box)
- satisfying click sounds and haptics, all adjustable or mutable
- a very cute bird (with a few hidden tidalBYTE easter eggs)

## How we built it

```
check-in (text / voice)
   │
   ├─ 1. Safety ....... Nemotron Safety Guard 8B  (+ local regex that works even if the API is down)
   │                    distress → 988 / counseling message, no coaching
   ├─ 2. Diagnose ..... Nemotron 3.5 Super VL 120B (reasoning)  → 4 lever scores, insights, tips, task, confidence
   │                    grounded in the student's words + their syllabus deadlines; strict JSON, validated, repaired
   ├─ 3. Choose ....... per-student feedback loop: which tricks actually got THIS student going?
   ├─ 4. Act .......... Nemotron 3.5 Lightning 30B (fast, tool calling) → make_first_step, create_calendar_block (.ics),
   │                    schedule_checkin, link_to_goal; self-corrects if it skips the tool; guaranteed fallback
   └─ 5. Learn ........ "Check in: <task>" → success / still stuck → updates strategy scores
syllabus PDF → pypdf → Nemotron Super (thinking off) → every graded item + due date/time + weight → steps 2 and 4
```

**Stack:**
- **Python + Streamlit** for the app, with a hand-built design system (custom CSS, inline SVG mascot, Web Audio sound synthesis, a small JS effects layer)
- **NVIDIA NIM API** through the OpenAI-compatible client
- **pypdf** for syllabus text
- **Local JSON** for check-ins and learning data

| File | What it does |
|---|---|
| `core/nemotron.py` | API client: retries on 5xx/network errors, fails fast on 4xx, JSON extraction + repair, reasoning on/off control, falls back to `reasoning_content` |
| `core/safety.py` | Safety gate (Nemotron Safety Guard + local regex) |
| `core/diagnose.py` | Temporal Motivation Theory diagnosis prompt, output validation, keyword backup |
| `core/agent.py` | The agent: safety → diagnose → choose → tool-calling loop with self-correction and a guaranteed-action fallback |
| `core/tools.py` | The tools Nemotron can call, plus their JSON schemas |
| `core/feedback.py` | The per-student learning loop |
| `core/syllabus.py` | Syllabus PDF → structured deadlines |
| `core/science.py` | Strategy library: every trick tagged with its discipline, citation and ADHD note |
| `core/week.py` | Calendar views, 12/24-hour times, reminder ↔ assignment matching |
| `core/ui.py`, `core/fx.js`, `core/mascot.py` | Design system, sounds/haptics/sparkles/focus sounds, Pip in 7 moods |
| `eval/` | Labeled eval set + runner (results below) |

## How Pip uses NVIDIA Nemotron (Best Use of Nemotron)

**What Nemotron does:** Nemotron *is* the agent. It isn't a chat box bolted on. Three Nemotron models each do a different job:

| Model | Job | Why this model |
|---|---|---|
| `nvidia/nemotron-3.5-super-vl-120b-a12b` | Diagnoses *why* the student is stuck (scores, insights, tips), and reads syllabi into structured deadlines | Accuracy matters most here, so we use the reasoning model with thinking on for diagnosis and off for extraction |
| `nvidia/nemotron-3.5-lightning-30b-a3b` | Calls tools that do real work: first steps, calendar blocks, check-ins, goal links | Fast, cheap and reliable at function calling |
| `nvidia/llama-3.1-nemotron-safety-guard-8b-v3` | Screens every single check-in before anything else happens | Purpose-built safety classifier, so the coaching never runs on a student in crisis |

**Why it matters:** students don't need another chatbot that says "you've got this." They need something that understands *which* problem they have and then removes friction, by actually creating the first step and the calendar block. Nemotron makes the understanding (reasoning), the doing (tool calls) and the safety (screening) possible in one pipeline.

**How we got the most out of it:**
- **Right model for each job.** Deep reasoning for diagnosis, a fast model for many tool calls, and a dedicated safety model. We control reasoning per call: thinking **off** for syllabus extraction cut it from ~18s with an *empty answer* to **~3s with all 7 items**, while diagnosis keeps thinking **on**.
- **Prompting for useful output, not filler.** The diagnosis prompt bans restating ("you said stuck, so you're struggling"). It asks for a non-obvious insight written to the student and a ≤12-word tip per lever, with examples. Check-in nudges are capped at 12 words, since the app shows the countdown.
- **Grounding.** Diagnosis sees the student's own words *and* their real deadlines from the syllabus. Scheduling sees the current time and the student's best focus time. Reminders are matched back to the assignment they belong to.
- **Structured, validated output.** Strict JSON schemas. Scores are clamped and validated, malformed JSON gets one repair turn, and a backup diagnosis is used if a model fails, with a warning shown in the UI rather than failing silently.
- **Feedback loops at two levels:**
  - **Inside one run:** if the fast model answers without calling the required tool, Pip tells it once to call it (self-correction), then falls back to a guaranteed action.
  - **Across runs:** every "It's going well! / Still stuck" updates a per-student strategy record, so Nemotron's next plan uses a trick that has actually worked for this student.
- **Explainable by design.** Every result has an "Agent trace" showing each Nemotron call, its latency, the tool calls with arguments, and the diagnosis JSON.
- **Measured, not vibes.** See the eval below.

### Eval: Nemotron's output quality, measured

`python eval/run_eval.py` runs a hand-labeled set against the live models (full per-case results in [`eval/results.md`](eval/results.md)):

| Test | Result |
|---|---|
| Diagnosis: right main lever (16 check-ins, 4 per lever) | **16 / 16** |
| Diagnosis: valid structured JSON | **16 / 16** |
| Safety: real distress routed to support | **4 / 4** (2 caught by the Nemotron safety model alone, without the regex) |
| Safety: false alarms on ordinary stress ("this exam is going to kill me lol") | **0 / 2** |
| Acting: fast model called the required tool itself (8 end-to-end runs) | **8 / 8** (1 needed the self-correction nudge, 0 needed the hard fallback) |
| Acting: scheduled times in the future / nudges ≤ 15 words | **8 / 8** / **8 / 8** |
| Median latency: diagnosis / full acting step | 9.0 s / 15.5 s |

The first eval run caught a real problem: the fast model replied in text without calling its tool in 1 of 4 runs. That's why we added the self-correction step, which took it to 8/8. The summary also appears inside the app as **Pip's report card** (in "The science behind Pip").

## Why it's interdisciplinary

Motivation isn't a computer-science problem. It's a human one. Every trick in Pip is labeled in the UI with the field it comes from:

| Field | What it gives Pip |
|---|---|
| **Psychology** | Temporal Motivation Theory (the diagnosis itself), self-efficacy (tiny first steps), implementation intentions (if-then plans) |
| **Behavioral Economics** | Present bias, commitment devices (self-imposed mini-deadlines), temptation bundling |
| **Cognitive Science & ADHD research** | Delay aversion and time blindness: why Impulsiveness and Delay are the loudest levers for ADHD brains, and why Pip makes time *visible* (countdowns, calendar blocks) |
| **Education / Learning Sciences** | Utility-value interventions ("why this matters to you"), syllabi as real input data |
| **HCI & Affective Design** | The "kawaii" effect (cute imagery improves careful, focused performance), shame-free tone, dyslexia-friendly type, calm motion you can switch off |
| **Public Health** | Safety-first routing to 988 and campus counseling before any coaching |
| **Computer Science / AI** | The Nemotron agent: reasoning, tool use, safety, evaluation and adaptation |

## Challenges we ran into

- **API access.** Our first Nemotron calls returned HTTP 403 (an account permission issue on the hosted endpoints). We built a full demo mode so UI work never blocked on it, then got the real models working.
- **The reasoning model "thought" its whole answer away.** On long syllabi, Nemotron Super's hidden reasoning used the entire token budget and returned *empty* content, so users saw "model did not return valid JSON". We fixed it by turning reasoning off for extraction (`chat_template_kwargs.enable_thinking=false`), raising the budget, and falling back to `reasoning_content`.
- **Useful vs. obvious AI output.** Early diagnoses just restated the student ("'stuck' suggests uncertainty"). We rewrote the prompt around non-obvious, second-person insights plus one tiny tip, and tested it on real messages.
- **Tool-calling reliability.** The fast model sometimes answered in prose instead of calling a tool. Our eval caught this, and the self-correction step plus guaranteed fallback means the student *always* leaves with a concrete action.
- **Safety without over-triggering.** Distress has to be caught every time, but "this exam is going to kill me lol" must not trigger a crisis message. We layered a local regex (works offline) with Nemotron Safety Guard and tested both directions.
- **Designing for ADHD brains.** Every feature had to earn its place. We cut things (a "big life goal" field, a separate science tab, a connection panel) because they added clutter without helping anyone get unstuck.
- **Streamlit, pushed hard.** Sounds, haptics, focus-sound synthesis, sparkles, Enter-to-send and the easter eggs all live in a small JS layer. Streamlit silently drops any script that contains HTML-tag-like text (even in a comment), which cost us an hour.

## Accomplishments that we're proud of

- A **real agent**, not a chatbot: safety → reasoning → choice → action → learning, across **three Nemotron models**, every step visible in the agent trace.
- **16/16** on our diagnosis eval, **4/4** distress caught with **0** false alarms, and **8/8** tool calls after adding self-correction. We measured it instead of guessing.
- A **feedback loop that changes behavior per student**, explained in plain words ("helped 0 out of 1 times, so let's try…").
- **Seven fields, one buddy**: every trick is tagged with its discipline and citation.
- **Accessibility as a feature**:
  - Lexend type, ≥17px text, roomy spacing, no italics or all-caps, AAA-contrast text
  - an "Easier reading" mode
  - reduced-motion support and a "Stop animations" switch
  - light and dark mode
  - adjustable or mutable sound and haptics
- A UI people actually *want* to open: color-coded everything, a calendar that reads like a planner, and Pip, who flaps, blinks, follows your cursor, and blushes when you tap him.

## What we learned

- Separating **understanding** (reasoning model), **doing** (fast tool-calling model) and **protecting** (safety model) produces better, faster and safer output than one big prompt.
- Reasoning models need budget and control: knowing when to turn thinking *off* matters as much as turning it on.
- **Evals change what you build.** One small labeled set found the tool-skipping bug that demos never showed us.
- For ADHD users, removing friction beats adding features. The most helpful thing Pip does is make the *first ten minutes* concrete.
- Behavioral science gives you a design spec: Temporal Motivation Theory told us exactly what to measure and what to fix.

## What's next for Pip: Your Tiny Study Buddy

- **Real notifications**: push or text Pip's check-ins at the scheduled time, and sync calendar blocks directly with Google or Apple Calendar.
- **Nemotron Parse** for scanned or table-heavy syllabi, and **voice check-ins** through Nemotron speech models (the hook is already in `core/transcribe.py`).
- **A bigger strategy library** retrieved with Nemotron Embed + Rerank, so Pip can match from hundreds of evidence-based tricks.
- **A bigger, student-written eval set**, and a study with students with ADHD to see whether diagnosed fixes beat generic advice.
- **Body-doubling rooms**: study next to a friend (or Pip) live.
- **Campus integration**: connect students to their disability resources office and counseling center.

## Run it

```bash
git clone https://github.com/rdyil/pip-tidalbyte-2026.git
cd pip-tidalbyte-2026
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # add your NVIDIA API key (build.nvidia.com) + model IDs
streamlit run app.py
python eval/run_eval.py     # optional: re-run the Nemotron eval (writes eval/results.md)
```

No API key? Pip runs in **demo mode** (keyword diagnosis and a sample syllabus) so the whole UI still works. A sample syllabus is in `samples/sample_syllabus.pdf`.

## Safety, privacy and honesty

- Pip is a **study tool, not a medical treatment** for ADHD, and it never diagnoses conditions.
- Every check-in is screened before coaching, and real distress gets a human-support message (988, campus counseling, 911), never a study plan.
- Check-ins and learning data stay in local JSON files (`data/`). Check-in text and syllabus text are sent to NVIDIA's hosted Nemotron API to be processed.

## References

- Steel, P. (2007). The nature of procrastination. *Psychological Bulletin.*
- Gollwitzer, P. M. (1999). Implementation intentions. *American Psychologist.*
- Bandura, A. (1977). Self-efficacy. *Psychological Review.*
- Ariely, D., & Wertenbroch, K. (2002). Procrastination, deadlines, and performance. *Psychological Science.*
- Milkman, K. L., Minson, J. A., & Volpp, K. G. M. (2014). Holding the Hunger Games hostage at the gym (temptation bundling). *Management Science.*
- Barkley, R. A. (1997). Behavioral inhibition, sustained attention, and executive functions. *Psychological Bulletin.*
- Sonuga-Barke, E. J. S., et al. (1992). Hyperactivity and delay aversion. *Journal of Child Psychology and Psychiatry.*
- Hulleman, C. S., & Harackiewicz, J. M. (2009). Promoting interest and performance in high school science classes. *Science.*
- Nittono, H., et al. (2012). The power of kawaii. *PLoS ONE.*
- Zajonc, R. B. (1965). Social facilitation. *Science.*

---

*Made with love by two ADHD brains at [tidalBYTE '26](https://f26.tidaltamu.com/). Psst: Pebble is hiding somewhere in the app.*
