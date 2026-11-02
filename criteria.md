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

## 1. Overall accuracy

Overall accuracy on the held-out test split is **at least 0.60** — on every
seed I test (42, 7 and 2024), not on average.

**Why this target:**
A model that always says `reaction`, my biggest label (about 44% of
`labels.csv`), already scores about 0.44, and guessing at random scores 0.33.
0.60 is where it's clearly sorting posts instead of leaning on the big label.
With about 43 posts in the test split that's 26 right. I didn't go higher
because the labels themselves aren't cleaner than about 80%: an independent
model applying my exact rules agreed with my pre-labels on 209 of 263 (79%).
Asking the classifier for 0.80 would be asking it to beat my own label noise.

---

## 2. No label gets ignored

Macro F1 on the test split is **at least 0.55** on every seed.

**Why this target:**
Accuracy can look fine while one label never gets predicted at all, and with
`analysis` at only 18% of the data that's the likeliest way this goes wrong.
Macro F1 counts all three labels equally, so a model that never says
`analysis` loses a third of the score. 0.55 is a bit under my accuracy target
because the smallest label pulls it down.

---

## 3. `analysis`, the label that matters most

F1 for `analysis` is **at least 0.50** on every seed.

**Why this target:**
`analysis` is the reason TakeMeter exists — it's the comment worth reading —
and it's my smallest label, about 52 posts, so only about 8 land in the test
split. One wrong post moves its F1 by roughly 0.1, which is why this is looser
than criterion 2. I also expect a specific failure here: most `analysis` posts
have numbers in them, but so do reactions like *"Fuckin Celtics are 40-20
wow."* If the model just learned "has digits → `analysis`", this is the
number that shows it.

---

## 4. Confident predictions are better ones

On every seed, the model's most-confident third of test predictions is **at
least 0.15 more accurate** than its least-confident third
(`accuracy_most_confident_third` minus `accuracy_least_confident_third` in
`results.json`).

**Why this target:**
TakeMeter is only useful as a filter if a confident "this is `analysis`" can
be trusted more than an unsure one. Each third is about 14 posts, so one post
is about 0.07. A gap of 0.15 is about two posts' worth. Anything smaller I
couldn't tell apart from luck.

---

## 5. Someone else's rules, applied the way they meant them

When I label the 30 staff posts under the staff taxonomy in unit 6, I agree
with the staff labels on **at least 22 of 30 (73%)**.

**Why this target:**
The staff note says a rate in the 70s is normal, because several of the 30
were picked for being ambiguous. Under my own rules, on my own posts, an
independent labeller agreed with me 79% of the time. Applying rules I didn't
write, on posts chosen to be hard, should come out lower than that, and 22 of
30 is the lowest I'd still call consistent.

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
