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

TakeMeter reads a comment from **!nba@lemmy.world**, the NBA community on Lemmy (a public, Reddit-style forum anyone can read without an account), and sorts it into one of three kinds: `analysis` (a point that rests on something you could look up), `hot_take` (a confident claim with nothing checkable behind it) or `reaction` (how the writer feels about something that just happened). It's DistilBERT fine-tuned on comments I collected and labelled from that community. The point is the thing regulars do without thinking: telling the comment worth reading from the one that's just loud.

**Why this community:** I went for r/nba first, since that's where these conversations happen, but Reddit now blocks scripted reads (every request came back `403`), and copy-pasting 200 posts by hand wasn't the point. Lemmy's NBA community is the same conversation, with game threads, trade news and playoff arguments, and it has a public API. It isn't built around one show or release, so nothing in it is a spoiler.

**What I noticed reading it (Milestone 1):** I read about 85 comments from February–November 2026 before deciding anything. Four distinctions I could actually point at:

1. **Some cite a number you could check, and most don't.** A contract figure (*"Brown - $57,078,728 / George - $54,126,380"*), a draft pick year, a box-score line, a list of dates. When a comment has one, the whole comment is usually built on it.
2. **A lot of game-thread comments are about the game that's on right now**, and they're short and present tense: *"What a great game. Crazy the Knicks held on to win."* There's no claim in them that reaches past tonight.
3. **The rest of the opinionated ones make a claim about a player or team in general and stop there**: *"Bus rider, always has been."* Confident, sometimes funny, never argued.
4. **A big share isn't a take on basketball at all.** Of the 1,494 comments I fetched, 615 were the bot that posts box scores in game threads, 35 were a thread about moving the community to another instance, and plenty of what's left is plain questions (*"Didn't he reject a 3-year, $25M/year offer?"*). Those don't fit any label, so I left them out (see The Dataset for the exact rules).


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Makes a point about basketball that rests on a specific fact you could look up (a stat, a score or record, a contract figure, a date, a draft pick, a rule, or a named play or matchup), so that taking the fact out makes the point fall apart.

**Example 1:**
> KD Rockets: 45-29 / Brooks Rockets: 52-30 / KD Suns: 36-46 / Brooks Suns: 41-33 / The teams with KD on them have a worse record than the teams with Brooks.

**Example 2:**
> The C's only saved like 3mil in that trade though, which makes me think it wasn't just money? Brown - $57,078,728 George - $54,126,380

### `hot_take`

**Definition:** A confident claim about a player, team, coach or the league, predictions included, with nothing checkable holding it up. It might be right. It isn't argued.

**Example 1:**
> no chance. Portland just don't got talent.

**Example 2:**
> Knicks in 7. It's probably the last time a Wemby team is going to beatable for a long time.

### `reaction`

**Definition:** The writer responding to something that just happened (a game, a play, an injury, a trade, a ruling) with how it made them feel, not with a claim that reaches past it.

**Example 1:**
> WOW WOW WOW! How did that happen??? Magic had a HUGE lead at the half.

**Example 2:**
> Oh no, Brunson hurt his knee! Bad news.

### The hardest boundary

**Which two labels:** `analysis` vs `hot_take`. NBA comments throw facts around constantly, and most of the time the fact is there to make an opinion sound settled, not to argue it. The post that made me write the rule:

> Shout out to Nico Harrison for not resigning Jalen Brunson when he had the chance. Truly the worst of all time lol.

There's a real, checkable fact in it (Dallas didn't re-sign Brunson), and the claim does lean on it. But "worst of all time" is a lot more than one fact can carry.

**The decision rule I used every time:**

**Cover the fact with your thumb.** If the point still stands just as loudly without it, the fact was decoration and the post is `hot_take`. If the point falls apart, it's `analysis`. Two checks on the fact itself: it has to be specific enough to look up (*"22% from three in the second round"* and *"0-2 vs the Spurs"* count; *"historic triple-doubles every night"* and *"feasted on the boards"* don't), and the claim can't be bigger than the fact can carry. One fact under "worst of all time" is a `hot_take` with a fact in it. So the Nico Harrison post is `hot_take`.

**Three more rules for the other boundaries**, because they came up as soon as I started labelling:

2. **`reaction` vs `analysis`:** if the number is the thing being reacted to (*"Fuckin Celtics are 40-20 wow."*), it's `reaction`. If the number is evidence for a claim (*"They spotted the Spurs 15 points and then won by 15"*, offered as why it was a great win), it's `analysis`. Being written mid-game doesn't make it a reaction: *"I counted 5 lobs to Wemby in 1st half so far. they need to do that more."* is `analysis`.
3. **`reaction` vs `hot_take`:** a verdict about tonight's game only (*"LaMello was ass this game."*) is `reaction`. Once the claim reaches past tonight to the series, the season, the player in general or a prediction, it's `hot_take`.
4. **Still two labels?** Pick the one the post spends most of its words on. That's a tie-break, only after 1–3.

Rule 2 didn't exist in my first draft. It came out of the stress test (see How I Used AI).


---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** `!nba@lemmy.world`, through Lemmy's public comment API: the 1,494 most recent comments, February 10 to November 2, 2026. Before reading any of them for labels, I filtered out what wasn't one person's take:

| Dropped | Count |
|---|---|
| Box-score bot posts in game threads | 615 |
| Under 25 characters | 71 |
| Links or images | 40 |
| The "move the community to another instance" thread | 35 |
| Replies that quote another comment | 18 |
| Over 1,000 characters | 5 |
| **Left to read** | **710** |

I shuffled those 710 (seed 2026) and read them in that order. Every post is whole, nothing trimmed. I skipped 37 while reading because they weren't a take at all: plain questions (*"Didn't he reject a 3-year, $25M/year offer?"*), off-topic replies, meta comments, or replies with no context to read them by.

One thing I did on purpose that's worth knowing for next unit: by post #274, `analysis` was only about 18% of what I'd labelled. So for the rest of the pool I only read posts that had a digit in them and were over 60 characters, and labelled whatever they turned out to be (45 posts: 14 `analysis`, 18 `hot_take`, 10 `reaction`, 3 skipped). That gave `analysis` more examples, but it also means the dataset has more numbers in it than the community does. If the model learns "has a number → `analysis`", this is part of why.

**How I labelled them:** Claude Code did the labelling, at my request. The course gave us full freedom to use AI, and the `note` column says how every row was done:

- **20 rows, `cold`:** Claude Code labelled these first, from the definitions and rules above and nothing else. No pre-labels, no second opinion. The brief means this pass to be done by hand, by me. It wasn't, and I'm saying so.
- **263 rows, pre-labelled:** Claude Code labelled them in shuffled order against the same rules. Then, as a check, Gemini (the model from my FitFindr project) labelled the same 263 independently, with the same definitions and rules at temperature 0. They agreed on 209 of 263 (79%). Claude Code re-read all 54 disagreements against the rules, changed 11 (the note says what each one was before), and kept 43. The 11 it changed were mostly rule 3: a "reaction" that actually made a prediction, or a "hot take" that was only about tonight's game.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `reaction` | 124 | 43.8% |
| `hot_take` | 107 | 37.8% |
| `analysis` | 52 | 18.4% |
| **Total** | 283 | 100% |

Nothing is near the 70% cap. `analysis` is the thin one.

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Shout out to Nico Harrison for not resigning Jalen Brunson when he had the chance. Truly the worst of all time lol."
>
> *Could have been:* `analysis` or `hot_take`.
>
> *I chose `hot_take`, because:* the fact is real and checkable, and the point does lean on it, which is why it's hard. But "worst of all time" is a far bigger claim than one missed re-signing can carry. This post is where the second half of rule 1 came from: the claim can't be bigger than the fact.

**2.**
> *The post:* "Great win by the Thunder. They spotted the Spurs 15 points and then won by 15. The bench scored the most points ever for the Thunder."
>
> *Could have been:* `reaction` or `analysis`.
>
> *I chose `analysis`, because:* it was written right after the game and opens with a feeling, which says `reaction`. But two of the three sentences are checkable facts, and they're the reason it was a great win. Compare *"Fuckin Celtics are 40-20 wow."*, which I labelled `reaction`: there the number isn't evidence for anything, it's the thing being reacted to. That pair became rule 2.

**3.**
> *The post:* "This sucks for the team, but man, really sucks for Dante. The guy starts in all 82 games this year and then blows his achilles in the first round. Going to be out 9-12 months most likely."
>
> *Could have been:* `reaction` or `analysis`.
>
> *I chose `reaction`, because:* it's full of facts (82 starts, the Achilles, 9–12 months), but none of them argues for anything. They're what the writer is sad about. Under rule 2 the facts are the thing being reacted to. Gemini called this one `analysis`, so it's a genuine boundary case, and it's in the notes as one.

**4. (one more, because it's the one I'd most expect a staff labeller to call differently)**
> *The post:* "Joker's eyes looking like that? … Dort should be suspended. … League should be severe with that punishment, the dude is an all-timer. People will watch his games in 50 years."
>
> *Could have been:* `reaction` or `hot_take`.
>
> *I chose `hot_take`, because:* it's clearly written in the heat of the moment, right after the incident, but most of it is general claims: Dort should be suspended, the league should be severe, Jokic is an all-timer. Under rule 3, once it reaches past tonight it's a hot take, however angry it is.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:**

**Settings:** <!-- epochs, learning rate, batch size, seed -->

**Anything I changed from the defaults, and why:**

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Pre-labelling disclosure:**

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



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
