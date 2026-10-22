# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter reads comments from [r/formula1](https://www.reddit.com/r/formula1/),
the main Formula 1 subreddit, during the back half of the 2026 season. Every
race weekend the post-race threads fill up with thousands of comments, and
they're a mix of people arguing from lap times and points maths, people
making confident calls with nothing behind them, and people just reacting to
what they watched. My classifier sorts a comment by *what kind of take it is*:
`analysis` (argued from a checkable fact), `hot_take` (a confident claim with
nothing holding it up), `reaction` (how the writer feels about something that
just happened), or `banter` (the joke is the whole point).


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Argues a point about the sport from a specific, checkable fact
(lap times, gaps, points, tyre stints, results, a past season) where the fact
is the reason for the conclusion.

**Example 1:**
> The narrative won't stop because people just love to feed off it but I'll
> debunk the "Lewis should've been let past when Charles came out of the pits
> on mediums" narrative.
>
> Lewis' lap times in free air before Charles came out of the pits:
>
> 37.2 37.1 37.0 37.2
>
> Charles lap times after his out lap on the mediums:
>
> 37.3 36.8 36.9 36.9
>
> It's quite obvious that Lewis didn't have any extra pace at that stage of
> the race to warrant a swap.

**Example 2:**
> A lot of people are saying Hamilton overruled Ferrari and rightfully so. But
> that's plain wrong. Leclerc was around 5s behind, pitted to inters, changed
> his wing, and still overtook Hamilton and was building a gap. The SC
> eventually neutralized things and made it seem like vetoing Ferrari was the
> correct decision. But looking at the process and not the outcome, Hamilton
> should've come in as instructed.

### `hot_take`

**Definition:** A confident, general claim about a driver, team, track or the
sport that offers no support, or only a fact too small to carry it.

**Example 1:**
> LIAM LAWSON WOULD DESTROY EVERYONE IN EQUAL CARS

**Example 2:**
> Mekies walked into Horner's well oiled machine and while has so far shown
> nothing significant on his own, he keeps the Verstappen's happy, and
> Mintzlaff only cares about that.

### `reaction`

**Definition:** How the writer feels about something they just watched or that
just happened to them: about their own experience of the moment, not a claim
about the world.

**Example 1:**
> The crowd when Lewis passed Hadjar was insane 😯

**Example 2:**
> Glad I gave up and went to bed, otherwise I'd have been knackered all day.
> But, my goodness, what a race.

### `banter`

**Definition:** A post whose point is the joke (a bit, a pun, a meme, an
absurd scenario) rather than a sincere claim or feeling.

**Example 1:**
> "Your vehicle is not subscribed to Wet Mode. Please insert credit card to
> enable."

**Example 2:**
> They were running windows and it did an update without permission

### The hardest boundary

**Which two labels:** `analysis` vs `hot_take`. r/formula1 loves a stat, and
half the confident takes have a number in them somewhere. *"Alonso in 8th is
why he keeps getting contracts"* has a checkable result in it and is still a
take.

**The decision rule I used every time:** It's `analysis` only if the fact is
checkable **and the conclusion is no bigger than what the fact can show.** A
set of lap times can show who was faster in that stint. Points maths can show
who can still win the title. If one result or one number gets stretched into a
verdict about a driver's talent, a team's culture or a whole season, it's a
`hot_take` with a stat in it. A heated tone doesn't make an argued post a
`hot_take`, and a number doesn't make an unargued one `analysis`.

**The other boundaries, so I'm not deciding them on the fly:**

- **`reaction` vs `hot_take`: who is the sentence about?** If it's about what
  just happened to me (what I watched, how it felt), it's `reaction`, even with
  an insult in it. If it's about how the world is in general, it's `hot_take`.
  *"That was so boring I fell asleep"* is `reaction`. *"Singapore is a boring
  track"* is `hot_take`.
- **`banter` vs everything else: take the joke out.** If a sincere claim is
  still standing underneath, label the claim. Sarcasm that's really an opinion
  (*"IT TOOK MAX 10 ATTEMPTS… WASHED MAN"*) is a `hot_take`. It's only `banter`
  when nothing sincere is left.
- **Out of scope, not collected:** questions (*"Why didn't Red Bull have this
  car at the start of season?"*), neutral explanations or news relays with no
  opinion in them, and replies that can't be understood without the comment
  above them. They aren't takes, and forcing them into a label would blur all
  four.

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Top-level comments from r/formula1 between
late August and 14 October 2026, pulled from the Arctic Shift Reddit archive
(Reddit's own API blocks scripted requests). Most come from the race-weekend
threads for the Dutch, Italian, Spanish, Azerbaijan, Malaysia (the relocated
Bahrain GP) and Singapore rounds (pre-race, post-qualifying, post-sprint,
post-race and day-after debrief), plus two Monday trash talk threads and three
news threads. Real stat-backed arguments barely show up in top-level comments,
so to reach 45 `analysis` posts I also searched the whole 2026 season for
comments containing phrases like "stint", "points behind" and "on average",
and kept the ones that make sense without the comment above them. That search
also turned up some of my `hot_take`s, the ones with a number used as
decoration, so a number isn't a giveaway for `analysis`. I left out
questions, neutral explanations, link-only posts and duplicates (the
out-of-scope list in my taxonomy).

**How I labelled them:** All 200 were pre-labelled with AI against the
definitions and decision rules above. I then labelled 20 of them myself, 5
from each pre-label, without seeing the pre-labels (`cold_20.csv`). My labels
replaced the pre-labels for those 20, which are marked `cold` in the `note`
column. The other 180 are marked `pre-labelled`.

My cold labels matched the pre-labels on **11 of 20 (55%)**, and the misses
weren't random:

- **4 of the 9 were me calling something `reaction`** that the pre-label had
  as `hot_take`, `analysis` or `banter`. For example, *"Lando is a massive
  muppet… he's more than happy to ruin his and his teammate's race"* felt
  like a reaction to me because it was written straight after the crash. My
  own written rule says a claim about a driver is `hot_take` even with an
  insult in it, so in practice I'm labelling by tone more than my rule says I
  should.
- **2 were `analysis` → `hot_take`**, so my bar for `analysis` is higher than
  the pre-label's. *"The average finishing position for a Ferrari for the last
  10ish gp is 4.5… so Lewis finishing 5th or lower isn't out of the question"*
  didn't feel like an argument to me.
- **3 were `banter` vs `reaction` / `hot_take` swaps**, which is the sarcasm
  boundary.

I kept the dataset as it is rather than relabelling the 180 to match my cold
reading. That means the data mixes two readings of the `reaction` and
`analysis` boundaries, and I expect that to show up in the confusion matrix
next unit.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `reaction` | 59 | 29.5% |
| `hot_take` | 56 | 28.0% |
| `banter` | 43 | 21.5% |
| `analysis` | 42 | 21.0% |
| **Total** | 200 | 100% |

Every label sits inside the 20–35% band from criterion 3. One thing I already
know will matter: `analysis` posts are much longer (median 347 characters,
against 82–151 for the other three), because an argument from evidence takes
more words. The model could learn "long post = `analysis`" instead of what I
meant.

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Hulkenberg is genuinely the most consistent driver I've ever
> seen... His previous eight results are: P8, P7, P11, P11, P10, P12, P8, P9.
> No matter what race it is, it feels like he's almost always in contention to
> finish in the points..."
>
> *Could have been:* `analysis` or `hot_take`
>
> *I chose, because:* `hot_take`. The eight results are real and checkable,
> and they do show he's been consistently around the points lately. But the
> conclusion is "the most consistent driver I've ever seen", which is far
> bigger than eight midfield results can show. Under my rule, the conclusion
> can't be bigger than the fact, so it's a take with a stat in it.

**2.**
> *The post:* "0.1 seconds. Bottas doing everything he could to help Red Bull."
>
> *Could have been:* `hot_take` or `banter`
>
> *I chose, because:* `hot_take`. It reads like a joke, but if you take the
> joke out there's still a sincere accusation underneath: that Bottas let a
> Red Bull past on purpose. My rule says sarcasm that's really an opinion is a
> `hot_take`. And the 0.1 seconds is too small a fact to carry an accusation
> like that, so it isn't `analysis` either.

**3.**
> *The post:* "Christian Horner was at Redbull for 20 years. With 525600
> minutes per year and at an average of 5 minutes per serving he could have
> consumed 2,103,840 bowls of coco pops in that time."
>
> *Could have been:* `banter` or `analysis`
>
> *I chose, because:* `banter`. It's the most number-heavy post in the
> dataset, and the maths actually checks out. But nothing is being argued.
> Take the joke out and there's no claim left at all, which is exactly my
> test for `banter`. I kept it on purpose: it's the clearest example that
> numbers don't make something `analysis`.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`, the notebook's default.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42. Trained on **`cpu`** (Intel Mac, torch 2.2.2), which
`results.json` records too. The run was 27 training steps and took about a
minute and a half.

**Anything I changed from the defaults, and why:** Only `LABELS`, set to my
four labels: `["analysis", "hot_take", "reaction", "banter"]`. I kept every
hyperparameter at its default. My criteria were filed before training, and
tuning settings after seeing a score would just move the standard to meet
the result.

One thing I noticed in the training log (validation set, not test): the loss
barely moved. Training loss went 1.36 → 1.26 and validation loss 1.32 → 1.25
over the three epochs, where random guessing across four labels is about
1.39. With 139 training posts and batch size 16, three epochs is only 27
weight updates, so I think the model is under-trained, not just confused.
I'm leaving that for next unit's "fix one thing" step instead of changing it
now.

**Split sizes:** 139 train / 31 validation / 30 test, stratified.

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 29 | 7 | **6** |
| `hot_take` | 39 | 9 | 8 |
| `reaction` | 41 | 9 | 9 |
| `banter` | 30 | 6 | **7** |

`analysis` (6) and `banter` (7) both have fewer than 8 test examples, so
their F1 rests on a handful of posts and will swing between seeds. One post
is worth about 0.15 of F1 for them, which is why criterion 2 sets their bar
lower.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1: pre-labelling the dataset**

- *What I asked for:* For an AI assistant to collect about 200 r/formula1
  comments and pre-label them against my four definitions and decision rules,
  keeping 20 aside, unlabelled, so I could label them cold.
- *What came back:* 200 labelled posts balanced across the four labels,
  including posts deliberately chosen for the hard boundary (`hot_take`s with
  a stat in them). Then my 20 cold labels matched the pre-labels on only
  **11 of 20**.
- *What I changed:* My labels replaced the pre-labels on those 20, so 9 posts
  changed label. The pattern in the 9 showed my `reaction` boundary is wider
  than my written rule (I label by tone), and my bar for `analysis` is
  higher. I weighed relabelling the other 180 to my reading, but decided to
  keep them as they are and treat the 55% as a finding to test next unit,
  since relabelling now would hide exactly the inconsistency the agreement
  check is meant to catch.

**Moment 2: the first training run**

- *What I asked for:* To run notebook sections 1–5 with my labels and write
  `results.json` and `test_split.csv`.
- *What came back:* The run plus a read of the training log: validation loss
  only went from 1.32 to 1.25, barely below random guessing for four labels
  (~1.39), because 3 epochs on 139 posts is only 27 weight updates. The
  suggestion was that more epochs would probably help.
- *What I changed:* Nothing in the run, on purpose. My criteria were already
  filed, and raising the epochs after seeing a test score would be tuning to
  the result. I kept the defaults and wrote the under-training down as my
  first candidate for next unit's "fix one thing and re-run".

**Pre-labelling disclosure:** 180 of the 200 rows in `labels.csv` were
pre-labelled by an AI assistant and are marked `pre-labelled` in the `note`
column. I labelled the other 20 myself without seeing the pre-labels; those
are marked `cold`, and where we disagreed (9 of 20) my label is the one in
the file.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.267 | 0.433 | +0.167 |
| Macro F1 | 0.198 | 0.373 | +0.174 |
| F1 — `analysis` | 0.348 | 0.615 | +0.268 |
| F1 — `banter` | 0.444 | 0.000 | -0.444 |
| F1 — `hot_take` | 0.000 | 0.353 | +0.353 |
| F1 — `reaction` | 0.000 | 0.522 | +0.522 |

Both columns are the same 30 held-out posts from `test_split.csv`. The
baseline is `facebook/bart-large-mnli` zero-shot, given my definitions from
`label_definitions.txt`; the trained column is seed 42 from last unit. I also
ran the baseline once with bare label names and no definitions (not
committed): **0.367 accuracy, 0.235 macro F1**, which beats the run with my
definitions.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

Written before running the baseline for this write-up. To be upfront about
what I already knew: I'd seen my trained model's seed-42 accuracy (0.433) at
the end of last unit, and I'd also seen an earlier trial run of this same
baseline, which scored 0.267.

My prediction is that the baseline lands **about the same as my trained
model, within roughly 0.10 of 0.433**. My trained model barely moved off
random guessing (27 weight updates in total), so a large general model reading
my four definitions shouldn't need to do much to match it.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

My prediction was wrong: the trained model beat the baseline by 0.167,
outside the "within 0.10" I called. The baseline came out exactly where the
earlier trial run had it, so I should have trusted that run over my
reasoning about how under-trained my model was.

The gap is smaller than it looks, though. On 30 posts, +0.167 is 5 posts.
And against the bare-names baseline (0.367) the gap shrinks to 0.067, which
is 2 posts. So fine-tuning added something, but not a lot. The one-seed
trained number can't yet be told apart from a general model given nothing but
four label names.

The per-label rows matter more than the headline:

- **The baseline never predicted `reaction` and almost never predicted
  `hot_take`.** With my definitions it called 17 of 30 posts `analysis`. My
  `analysis` definition is the only one that literally says "about the
  sport", which every post on r/formula1 is, so the zero-shot model reads it
  as a match for nearly anything. That's a wording problem in my definition
  that I'd never have spotted by reading it myself. It also explains why my
  definitions did *worse* than bare names (0.267 vs 0.367).
- **`banter` is the one label where the baseline beat my model**: 0.444
  against 0.000. My trained model didn't get a single joke right on seed 42,
  while a general model reading "the joke is the whole point" got 4 of 7.
  So `banter` is learnable from the text; my model just didn't learn it in 27
  updates.
- **`reaction` and `hot_take` are where training actually paid off** (+0.52
  and +0.35). Those are the labels that depend on r/formula1 tone, which a
  general model reading definitions can't pick up but a few examples can.


---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Trained on `cpu`, torch 2.2.2, default settings (3 epochs, lr 2e-5, batch
16). Seed 42 reproduced last unit's run exactly (0.433).

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 on every seed | 0.433 | 0.533 | 0.567 | MISSED |
| 2. Per-label F1 | `hot_take`, `reaction` ≥ 0.55; `analysis`, `banter` ≥ 0.40 | ana 0.615 · hot 0.353 · rea 0.522 · ban 0.000 | ana 0.667 · hot 0.500 · rea 0.667 · ban 0.000 | ana 0.833 · hot 0.556 · rea 0.609 · ban 0.000 | MISSED |
| 3. Balance | largest ≤ 35%, smallest ≥ 20% | 29.5% / 21.0% | 29.5% / 21.0% | 29.5% / 21.0% | MET |
| 4. Staff agreement | ≥ 21/30, ≤ 3 `analysis`↔`hot_take` misses | 8/30 · 4 misses (labelled once, not per seed) | same | same | MISSED |
| 5. Confidence gap | most-sure third − least-sure third ≥ 0.20 | 0.70 − 0.10 = 0.60 | 0.70 − 0.30 = 0.40 | 0.60 − 0.60 = 0.00 | MISSED |

The notebook's spread table:

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.433 | 0.533 | 0.567 | 0.133 |
| Macro F1 | 0.373 | 0.458 | 0.499 | 0.127 |
| F1 — `analysis` | 0.615 | 0.667 | 0.833 | 0.218 |
| F1 — `hot_take` | 0.353 | 0.500 | 0.556 | 0.203 |
| F1 — `reaction` | 0.522 | 0.667 | 0.609 | 0.145 |
| F1 — `banter` | 0.000 | 0.000 | 0.000 | 0.000 |

Accuracy moved 0.133 across seeds, more than 10 points, so a single seed
can't tell me much on 30 test posts.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42:

| true \ predicted | analysis | hot_take | reaction | banter |
|---|---|---|---|---|
| **analysis** | 4 | 2 | 0 | 0 |
| **hot_take** | 2 | 3 | 3 | 0 |
| **reaction** | 1 | 2 | 6 | 0 |
| **banter** | 0 | 2 | 5 | 0 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

**5 real `banter` posts were called `reaction`, and 0 went the other way.**
The model didn't learn the `banter`/`reaction` boundary at all. More than
that, the `banter` column is empty on all three seeds: across 21 test-set
jokes, it never once predicted `banter`. It sends jokes to `reaction` (12 of
21 across seeds), `hot_take` (7) or `analysis` (2). The direction makes sense
for my data: most of my jokes are short, in-the-moment and written during a
race, exactly like my reactions, so on surface features a joke looks like a
reaction.

The other pairs look the way I hoped. `analysis` is the most stable label
(F1 0.62–0.83), and almost nothing gets confused with it except a couple of
`hot_take`s each way, which is my hardest boundary doing roughly what it
should.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.60 on every seed | **MISSED** | 0.433 / 0.533 / 0.567. Not one seed reached it. |
| 2 | Per-label F1 | `hot_take`, `reaction` ≥ 0.55; `analysis`, `banter` ≥ 0.40 | **MISSED** | `analysis` held on all three (0.62–0.83), but `banter` was 0.000 on every seed, `hot_take` missed on 42 and 7, and `reaction` missed on 42. |
| 3 | Balance | largest ≤ 35%, smallest ≥ 20% | **MET** | Counted from `labels.csv`: 29.5% / 21.0%. It's a property of the dataset, so it's the same on every seed. |
| 4 | Staff agreement | ≥ 21/30 and ≤ 3 `analysis`↔`hot_take` misses | **MISSED** | 8/30 (27%) from `agreement.py`, with 4 `analysis`↔`hot_take` misses. Both halves failed. |
| 5 | Confidence gap | most-sure third beats least-sure third by ≥ 0.20 | **MISSED** | 0.60 and 0.40 on seeds 42 and 7, but 0.00 on seed 2024. It has to hold on all three. |

None of my criteria turned out to be *broken*: every one gave me a number I
could call. So I haven't revised any; the four misses stay misses.

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**1. Accuracy (MISSED): mainly a model that never finished learning, plus
a labelling problem on `reaction`.**

- *Under-training.* Validation loss only fell from 1.32 to 1.25 over three
  epochs. Random guessing across four labels is about 1.39. With 139 training
  posts and batch size 16, three epochs is just 27 weight updates. The model
  stopped while it was still close to guessing.
- *How I know it's not just a hard task:* the zero-shot baseline, which never
  saw a single example, got 4 of 7 `banter` posts right from the definition
  alone. The signal is in the text; my model just never picked it up.
- *Part of it is my labels.* Half of the trained model's non-`banter` errors on
  seed 42 sit on `hot_take` ↔ `reaction` (5 of 10: 3 one way, 2 the other). My
  agreement report shows that's exactly the boundary I apply loosely: 13 of my
  22 staff misses were me over-using `reaction`, and 5 went the other way. My
  cold 20 last unit showed the same drift. When the person writing the labels
  can't hold a boundary steady, the model can't learn it either.
- *And the measure itself is shaky.* 30 test posts means one post is 3.3
  points, and accuracy moved 0.133 across seeds. Even a better model would be
  hard to call against a 0.60 line.

**2. Per-label F1 (MISSED): `banter` is an under-training problem;
`hot_take` is a labelling problem.**

- *`banter` scored 0.000 on every seed because the model never predicted it
  once.* The `banter` column of the confusion matrix is empty on all three
  seeds, and 12 of the 21 test jokes went to `reaction`. That's a class the
  model never learned, not one it learned badly. The zero-shot baseline
  scoring 0.444 on the same posts says the label is learnable. My jokes are
  short and written mid-race, like my reactions, so a barely-trained model
  falls back on surface features and they look the same.
- *Some of it is labelling too.* 3 of my 9 cold-20 disagreements were
  `banter` ↔ `reaction`/`hot_take` swaps. The staff set can't test this,
  because it has no `banter` label, so this is the one place my evidence is
  thinner.
- *`hot_take` (missed on 2 seeds) is the boundary I can't hold myself.* It's
  the label most often confused with `reaction`, and the agreement report
  puts my own `hot_take`/`reaction` judgement at the centre of my misses.
  That points at labelling, not the model.
- *It isn't "too few examples".* `analysis` is my smallest label (42 posts, 6
  in test) and it was my best on every seed (0.62–0.83). So the count isn't
  what's sinking `hot_take` (56 posts). One caveat against myself: `analysis`
  posts are also much longer (median 347 characters), so some of its success
  may be the model learning "long post" rather than "argued from a fact".

**4. Staff agreement (MISSED): a consistency problem in how I apply the
`reaction` rule.**

- 19 of 22 disagreements are pile (a): the staff rule covered the post and I
  applied it loosely. They aren't all one direction (13 into `reaction`, 5
  out of it), so I wasn't following a different rule of my own. I was going
  by tone instead of asking "is this about the writer or about the world?"
- The `analysis` half (4 misses, 2 of them flagged as hard by staff) is
  partly a real difference rather than sloppiness. My own taxonomy demands
  that "the conclusion is no bigger than what the fact can show", which is
  stricter than staff Rule 1. sp_005 and sp_010 are the pile (b) cases.
- I set 70% because staff said the 70s are normal. 27% is far enough below
  that I'm not going to blame the posts: it's how carefully I applied a
  written rule.

**5. Confidence gap (MISSED on one seed): the model was never confident about
anything.**

- Average confidence in its top choice was 0.30–0.34 on every seed, against
  0.25 for a uniform guess. On seed 2024 it was 0.301 when right and 0.300
  when wrong. Sorting posts into confidence thirds was sorting by differences
  in the third decimal place, so whether the "most-sure" third beat the
  "least-sure" third came down to the luck of the split: 0.60 on seed 42,
  0.00 on seed 2024.
- That's the same under-training as criterion 1. Confidence can't mean
  anything until the model has moved away from guessing.


---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 8 / 30 = 27%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.** `sp_001`
> *The post:* Pulled the splits from the last five editions. The winning time has moved by 40 seconds total in ten years, but the 100th-place time is four minutes faster. The depth changed, the front didn't.
>
> *Staff said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* Staff. Two checkable numbers, and the conclusion ("the depth changed, the front didn't") is exactly what their contrast shows. Rule 1 is satisfied: the fact carries it.
>
> *Which pile:* (a)

**2.** `sp_003`
> *The post:* The club championship scoring changed in 2019 from top-6 to top-10. Since then our B team has finished above the A team twice. That's not depth improving, it's the scoring rewarding turnout.
>
> *Staff said / I said:* `analysis` / `reaction`
>
> *My call, and why:* Staff. A dated rule change and a counted result, used to reject a rival explanation. Nothing in it is about how the writer feels, so Rule 2 rules out `reaction`.
>
> *Which pile:* (a)

**3.** `sp_004`
> *The post:* Everyone blames the hill at 18 miles. I split the field into people who ran the first half under 1:45 and over, and the slowdown after 18 was almost identical for both groups. If the hill were the problem the fast starters would fall apart harder.
>
> *Staff said / I said:* `analysis` / `reaction`
>
> *My call, and why:* Staff. It sets up a test, reports the result and draws the inference. That's the clearest `analysis` in the set, and I have no rule-based case for `reaction`.
>
> *Which pile:* (a)

**4.** `sp_005`
> *The post:* Carbon plates aren't the whole story. My times dropped 2% in the shoes and another 2% the following year in the same pair, when the only thing that changed was going from 35 to 50 miles a week.
>
> *Staff said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* Staff, under Rule 1 as written: the two comparable numbers are what the claim rests on. I went `hot_take` because it's one runner with a confounded comparison. But Rule 1 only asks whether the fact carries the argument, not whether it's strong enough to, and staff flagged this one as hard.
>
> *Which pile:* (b): the rules have no threshold for how much evidence counts

**5.** `sp_006`
> *The post:* Checked the entry lists. 41% of this year's field have never run the distance before, against 26% five years ago. That's why the finish-line photos look different, not because standards dropped.
>
> *Staff said / I said:* `analysis` / `reaction`
>
> *My call, and why:* Staff. Two percentages, explicitly used to reject "standards dropped". That's Rule 1 straight.
>
> *Which pile:* (a)

**6.** `sp_007`
> *The post:* Ran the same loop in 4C and in 24C within a fortnight, same effort by heart rate. 90 seconds slower in the heat over six miles. People keep saying heat costs you 'a bit' and it's more than that.
>
> *Staff said / I said:* `analysis` / `reaction`
>
> *My call, and why:* Staff. It's the writer's own run, which is what pulled me to `reaction`, but Rule 3 says exactly that being personal or recent doesn't outrank a load-bearing fact. The 90 seconds is the argument against "a bit".
>
> *Which pile:* (a)

**7.** `sp_008`
> *The post:* Went back through the club results going back to 2012. Every single year the number of people who enter the spring marathon and don't start it sits between 18 and 23 percent. It's not a bad-luck year, it's the base rate.
>
> *Staff said / I said:* `analysis` / `reaction`
>
> *My call, and why:* Staff. A range across a named span, used to argue against a one-off explanation.
>
> *Which pile:* (a)

**8.** `sp_009`
> *The post:* The 10K route was remeasured in 2021 and came out 60 metres long. Everyone comparing PBs from before and after that is comparing two different races, which is about 12 seconds at club pace.
>
> *Staff said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* Staff. Date, measurement and the arithmetic consequence. The confident "everyone… is comparing two different races" tone isn't a reason for `hot_take` when the fact carries it.
>
> *Which pile:* (a)

**9.** `sp_010`
> *The post:* Three of us ran the same session on the same day, different shoes, within 10 seconds of each other over 5 miles. I'm not saying shoes do nothing, but whatever they do was smaller than the noise between three people.
>
> *Staff said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* Staff, under the rule as written: the observation is checkable and the hedged conclusion rests on it. My `hot_take` was about three people on one day being too small to show anything. That's the same gap as sp_005, and staff flagged it as hard too.
>
> *Which pile:* (b): same evidence-threshold gap

**10.** `sp_011`
> *The post:* Anyone who races more than once a month isn't training, they're just collecting medals.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff. A claim about a whole group with zero support, and nothing happened to the writer. Rule 2 decides it.
>
> *Which pile:* (a)

**11.** `sp_012`
> *The post:* Heart rate zones are astrology for runners.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff. Funny, but it's a pure general claim with nothing argued.
>
> *Which pile:* (a)

**12.** `sp_013`
> *The post:* The club needs to stop pretending the Tuesday session is for everyone. It isn't and it never has been.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff. Strong and general, with no support. The frustrated tone isn't a report of something that just happened.
>
> *Which pile:* (a)

**13.** `sp_014`
> *The post:* Times are down about 3% since the shoe rules changed, so obviously the shoes are the whole story and the sport has a problem.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff, and it isn't close. It's almost word for word the Rule 1 example of a stat used as decoration ("so obviously" does the work), and it's a claim about the sport, so Rule 2 rules out `reaction` too.
>
> *Which pile:* (a)

**14.** `sp_015`
> *The post:* Marathon training plans are all the same plan with different fonts.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff. An aphorism about the world with no support.
>
> *Which pile:* (a)

**15.** `sp_016`
> *The post:* If you need a gel before 90 minutes you have a fuelling problem, not an energy problem.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff. A confident prescription for everyone, with nothing behind it.
>
> *Which pile:* (a)

**16.** `sp_019`
> *The post:* Nobody actually enjoys the long run. They enjoy having done it, which is a completely different thing, and the sport would be more honest if it said so.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff. It reads reflective, which is why I went `reaction`, but "nobody actually enjoys" is a claim about everyone, not a report of the writer's morning. Rule 2 covers exactly this.
>
> *Which pile:* (a)

**17.** `sp_020`
> *The post:* Every injury thread on this forum is the same injury and the same three replies.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff. It sounds observational, but no count is given; the writer is asserting a general pattern, not reporting their own experience.
>
> *Which pile:* (a)

**18.** `sp_021`
> *The post:* Four minutes off. FOUR. I've been chasing that time for two years and I'm sat on the kerb outside the finish just grinning at nothing.
>
> *Staff said / I said:* `reaction` / `hot_take`
>
> *My call, and why:* Staff. The writer's own race, minutes ago, with feeling front and centre. The four minutes is the news, not a claim about anything.
>
> *Which pile:* (a)

**19.** `sp_025`
> *The post:* Blew up at 20 with the pace band still in my hand like it was going to help. Rough one.
>
> *Staff said / I said:* `reaction` / `analysis`
>
> *My call, and why:* Staff. "At 20" is a mile marker, not a fact that carries an argument; nothing is being argued at all.
>
> *Which pile:* (a)

**20.** `sp_027`
> *The post:* Wind was so bad on the out section I was leaning at about 20 degrees. Coming back felt like cheating.
>
> *Staff said / I said:* `reaction` / `hot_take`
>
> *My call, and why:* Staff. A report of the run just done; "felt like cheating" is about the writer's experience, not a claim.
>
> *Which pile:* (a)

**21.** `sp_028`
> *The post:* Third DNF this year. I don't really want to talk about it, just wanted to put it somewhere.
>
> *Staff said / I said:* `reaction` / `hot_take`
>
> *My call, and why:* Staff. "Third this year" is countable, but nothing is argued from it; it's the writer reporting their own state.
>
> *Which pile:* (a)

**22.** `sp_029`
> *The post:* Just realised the shoes I set every PB in this year are the ones I bought by accident in the wrong size.
>
> *Staff said / I said:* `reaction` / `hot_take`
>
> *My call, and why:* Staff, on the rules as written. My reading was that it's a wry dig at shoe obsession, in a set where three other posts argue about shoes. But Rule 2 asks what the post asserts, and on the surface it only reports the writer's own realisation. The staff rules have no way to handle a claim made by implication or irony. My own taxonomy needed a whole label (`banter`) and a "take the joke out" rule for exactly this.
>
> *Which pile:* (b): no rule for implied claims

**What the pattern in my disagreements tells me:**

Pile count: **19 (a), 3 (b), 0 (c).** I didn't win any of these on the
staff's own rules, and I'd rather say that than dress a loss up as a win. The
three (b)s are real gaps: Rule 1 has no threshold for how much evidence a
fact needs (sp_005, sp_010), and Rule 2 can't handle a claim made through
irony (sp_029).

The (a)s aren't random; they're the `reaction` boundary, in both directions:

- **13 of my 22 misses were me calling something `reaction`** that staff
  called `hot_take` (8) or `analysis` (5). Heated, reflective or first-person
  posts went to `reaction` even when they were claims about the world
  (sp_012, sp_014, sp_019) or arguments from the writer's own data (sp_007).
- **5 went the other way**: posts that really were the writer reporting what
  just happened to them (sp_021, sp_027, sp_028) got called `hot_take` or
  `analysis`.

Errors in both directions mean I wasn't consistently applying some different
rule of my own. I was applying Rule 2 ("about the writer or about the
world?") loosely and going by feel. And it isn't new: my cold 20 last unit
disagreed with the pre-labels the same way, with 4 of 9 misses being me
calling something `reaction`.

What this means for my own 200: two separate checks now say that **my
personal reading of `reaction` is the least reliable thing in my labelling**.
In my dataset that mostly affects the 20 cold rows, since the other 180 were
pre-labelled by applying the written rules. But that's a mix of two readings
of the same boundary, and the model's mistakes sit right on it: `banter` →
`reaction` (12 of 21 across seeds) and `hot_take` ↔ `reaction`. I'll use that
as evidence in the diagnoses that at least part of the confusion is a
labelling problem, not just a model problem.
---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
