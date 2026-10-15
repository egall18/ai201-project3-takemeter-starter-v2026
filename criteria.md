# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy of at least 0.60 on every seed

On the held-out test split, overall accuracy is **at least 0.60 on each of
seeds 42, 7 and 2024**, not on average.

**Why this target:** With four labels, a random guesser gets about 0.25, and
because criterion 3 caps my biggest label at 35%, always guessing the biggest
label gets at most 0.35. 0.60 is roughly 18 of 30 test posts right, the point
where I'd actually trust it to sort a post-race thread for me. I didn't go
higher because the model only trains on about 140 posts, and `banter` vs
`hot_take` comes down to sarcasm, which is hard for me to call, let alone
DistilBERT. With 30 test posts each one is worth 3.3 points, so a target of
0.70 would mostly be testing which seed I got.

---

## 2. Every label pulls its weight: F1 ≥ 0.55 for the big two, ≥ 0.40 for the thin two

On every seed: F1 for `hot_take` and `reaction` is **at least 0.55**, and F1
for `analysis` and `banter` is **at least 0.40**.

**Why this target:** `analysis` and `banter` will be my smallest labels.
Real stat-backed arguments barely exist outside post-race and debrief
threads, and `banter` shares its boundary with sarcastic `hot_take`s. At
around 40 posts each, only about 6 of each land in the test split, so one
miss moves their F1 by roughly 0.1, which is why their bar is lower. 0.40 is
still a real bar: a model that just ignores a label scores 0 on it.
`hot_take` and `reaction` are bigger and have clearer surface signals
("what a race", "I'm shaking" vs "X is overrated"), so I expect more from them.

---

## 3. Balance: no label above 35% of my data, none below 20%

In `labels.csv`, the largest label is **at most 35%** of rows and the smallest
is **at least 20%** (40+ posts out of 200).

**Why this target:** The brief caps any label at 70%, but with four labels
that would leave the other three at 10% each, about 3 posts per test split.
20% is the floor that gets every label to about 6 test posts, the minimum
where its F1 means anything. Left alone, r/formula1 is mostly jokes and hot
takes, so this forces me to go and get `analysis` from post-race and debrief
threads on purpose instead of letting the subreddit's natural mix decide. The
35% cap also keeps criterion 1 honest: guessing the biggest label can't get
close to 0.60.

---

## 4. Consistency: at least 21/30 with the staff set, and no more than 3 misses on the fact boundary

On the 30-post staff set, labelled under the staff taxonomy, I agree with staff
on **at least 21 of 30 (70%)**, and **no more than 3** of my disagreements are
`analysis` ↔ `hot_take`.

**Why this target:** Staff say several of the 30 are deliberately ambiguous
and that the 70s are normal, so 70% overall is my "I applied someone else's
rule like they did" bar. The second half is the part that tests me: staff rule
1 ("does the fact carry the argument?") is basically my own hardest-boundary
rule ("the conclusion is no bigger than what the fact can show"). If I
can't apply that rule cleanly on their posts, I can't have applied it cleanly
across my 200, and I'll know an `analysis`/`hot_take` mix-up in my confusion
matrix is a labelling problem, not a model problem.

---

## 5. Confidence means something: the most-sure third beats the least-sure third by 0.20

On every seed, accuracy on the model's **most-confident third** of test posts
is **at least 0.20 higher** than on its **least-confident third**
(`accuracy_most_confident_third` vs `accuracy_least_confident_third` in
`results.json`).

**Why this target:** The useful version of TakeMeter auto-tags the posts it's
sure about and leaves the unsure ones, which I expect to be mostly sarcasm,
for a person. That only works if confidence actually tracks being right. Each
third is about 10 posts, so 0.20 means at least 2 more right out of 10. A 0.10
gap is a single post, which I'd call noise, not a signal.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
