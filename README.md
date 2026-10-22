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
| 4. Staff agreement | ≥ 21/30, ≤ 3 `analysis`↔`hot_take` misses | — | — | — | pending (Milestone 3) |
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
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

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

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



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
