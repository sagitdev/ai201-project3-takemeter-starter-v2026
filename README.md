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

TakeMeter sorts IMDb user reviews by how the opinion is argued, not by whether
the writer liked the film. My data is 200 reviews from 10 films across five
genres, sampled evenly across star ratings. The classifier sorts each review
into `critique` (backs a verdict with something specific in the film),
`hot_take` (a verdict with nothing specific), or `experience` (mainly a
reaction the film caused, like crying or falling asleep).
---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

Each label holds both positive and negative reviews, so it describes how a
review is written, not whether the writer liked the film.

### `critique`

**Definition:** Backs a judgement of the film with something specific in it(an actor, character, director, scene, plot point, the ending, the style, another named film, or a filming or technical choice) and says what that thing is like or why, not just that it was good or bad.

**Example 1:** *(Resident Evil, positive)*
> As a Resident Evil games player since RE1, I approve this movie 10/10. … The tunnel scene in particular was brilliant. The camera angles are brilliantly game-like, and the environment lighting and score are perfectly on point.

**Example 2:** *(Obsession, negative)*
> I don't get the OBSESSION! Honestly I don't understand the hype. I felt the movie was repitive and predictable, with very little risk taken. The color was desaturated to the point it felt like a mistake. … It leaned way too hard on the in and out of reality displayed by Nikki, which was uninteresting and overdone. You could literally time a jump scare. …

### `hot_take`

**Definition:** States a verdict on the film without pointing to anything specific in it.

**Example 1:** *(The Shawshank Redemption, positive)*
> Movie worth watching. A very good movie. Worth watching.brilliant movie. One of the most memorable movies that you won't forget about.

**Example 2:** *(WALL-E, negative)*
> …。 Pretty dull and boring. And heavily overrated. I don't see what's so good about it.

### `experience`

**Definition:** Mainly reports a reaction the film caused in the writer, or in the people they watched with (crying, falling asleep, walking out), rather than judging the film, and points to nothing specific in it.

**Example 1:** *(Life Is Beautiful, positive)*
> Life is really beautiful. Just yesterday I saw the movie, basically am not a happy person. After I saw this movie I have learned to be happy with what I have. I saw his Oscar award videos before I saw the move. Then I understood. Thanks Robert

**Example 2:** *(Life Is Beautiful, negative)*
> Very disappointing. I watched this at home, and I found it tedious. I turned it off after 1/2 hour because I found it boring and Benigni's humor not funny. How this movie got selected for Best Picture nomination baffles me. Perhaps the story picked up after a while, but I wasn't around to see it!

### The hardest boundary

**Which two labels:** `critique` and `hot_take`. Most reviews mention some part of the film (the acting, the story, the CGI). The hard question is when a mention counts as specific.

**The decision rule I used every time:**

1. **`critique` vs `hot_take`: blank out the names, then look at what's left.**
   Replace any actor, character, director or film title with a placeholder ("the lead", "another film"). If all that remains is an evaluation word (great, bad, flat, boring, worse, too much, dragged), it is not specific. It is specific only if it describes what the thing does or gives a reason.
   One specific sentence is enough to make the review a `critique`, however heated the rest is.
   A description of what happens in the film (a particular scene, a character's situation) also counts as specific, even if the verdict on it is only an evaluation word.
   - "Good Actors, Good Director" → only evaluation words → `hot_take`
   - "Bad CGI, no sense of heights" → "no sense of heights" says what the CGI fails to do → `critique`
   - "A simple plot like this did not need a 2.5h movie" → a checkable fact → `critique`

2. **`experience` vs `hot_take`: only reactions count.** 
   A reaction the film caused counts (crying, laughing, falling asleep, walking out, turning it
   off). Where, how or with whom they watched (on a plane, at home, with my wife) is context and does not count. If a review has both a reaction and a verdict, I pick whichever takes up more of it.

3. **Order: rule 1 first.** 
   If it finds anything specific, the label is `critique` and I stop. 
   Only if it doesn't do I use rule 2.

Complaints about the rating or the hype ("Overrated", "Really? 10/10?") follow the same rules. I considered a fourth label for them: about 60 of my 200 reviews comment on the rating. It wouldn't hold, because a rating complaint is a topic, not a way of writing. "Overrated … How the wardens didn't notice he was chipping through the wall everyday to escape is beyond me" argues from a plot detail, while "Dont get why is this greatest here, lol ok" argues from nothing. They needed to be `critique` and `hot_take`, not one shared label.

**How the rule was tested:** 
I classified eight boundary reviews an AI wrote for me. I got 3 of 8 right on the first pass. The misses showed two leaks. I was counting a name as specific ("Ryan Gosling is great"), which led to the blank-out-the-names step. A single side remark about the viewing conditions ("watched this on a plane with terrible headphones") was turning a review into `experience`, which led to "only reactions count".

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

> **Status at the unit 5 deadline:** collection is done (200 reviews in
> `labels.csv`). Labelling is in progress: the 20 cold rows are labelled and
> checked; the other 180 are pre-labelled by AI and waiting for my
> review, so their `label` column is still empty. Training (Milestone 5) has not
> run yet. I'll finish both and commit them. The two worksheets are committed
> as they stand: `cold_20.md` holds my cold labels and re-check, and
> `prelabel_180.md` holds the AI's unchecked pre-labels.

**Where the posts came from:** IMDb user reviews, copied by hand. 20 reviews
from each of 10 films: Obsession, Resident Evil, Fall 2: Deadpoint, Project
Hail Mary, Spider-Man (2002), The Wandering Earth, The Shawshank Redemption,
Life Is Beautiful, Your Name, WALL-E. For each film I took reviews evenly
across star ratings and skipped any flagged as spoilers. The star rating is
not in the text; each row is the review's title followed by its body.

**How I labelled them:** <!-- Cold first? Pre-labelled with AI and corrected?
Say so plainly — the disclosure is required, not penalised. -->
I labelled 20 reviews cold first: two per film, no AI, no translation. 16 of my
20 cold labels were `critique`. When I re-checked them against my own
blank-out-the-names rule, 6 turned out to be only evaluation words and became
`hot_take`, leaving 10 and 10. Both labels are kept in the `note` column
(`cold: …; checked: …`). The other 180 were pre-labelled by AI using my
definitions and rules. I am reading and correcting every one.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
|  |  |  |
|  |  |  |
|  |  |  |
| **Total** |  | 100% |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

**2.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

**3.**
> *The post:*
>
> *Could have been:*
>
> *I chose, because:*

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

I used Claude (Claude Code) as a tutor throughout. These notes cover what I've
done up to the unit 5 deadline.

**Moment 1: drafting the labels**

- *What I asked for:* Draft two to four labels with one-sentence definitions,
  based on my 200 reviews and the eight distinctions I wrote down in
  Milestone 1 (`notes.md`).
- *What came back:* `critique`, `hot_take` and `experience`, each with a real
  positive and negative example. It also explained why it didn't use emotional
  intensity, and why the rating-complaint reviews didn't work as a fourth label.
  It flagged two risks: what counts as "specific", and `experience` being thin.
- *What I changed:* I widened `critique` to list more kinds of specific things
  (style, the ending, a named director, another named film). I added "points to
  nothing specific" to `experience` so it can't overlap with `critique`. I also
  added a "more than three sentences" condition to `critique`. The AI showed me a
  two-sentence review that would then fit no label, so I removed it. At first I
  said that any mention of acting or effects should count as `critique`. Then I
  saw that about 77% of reviews mention some part of the film, which would break
  the 70% cap, so I chose to require a description or reason, not just a mention.

**Moment 2: stress-testing the decision rules**

- *What I asked for:* I had the AI draft my decision rules. Then I asked for
  eight made-up reviews that sit on my boundaries. I classified them myself
  before seeing its answers.
- *What came back:* Eight boundary reviews. Measured against my own rules, I
  got 3 of 8 right on the first pass.
- *What I changed:* My misses showed I was treating any name as specific
  ("Ryan Gosling is great"). That became the blank-out-the-names step in rule 1.
  On one review I disagreed with the AI's answer. A side remark about watching on
  a plane with bad headphones was making it `experience`, and I argued that
  viewing conditions aren't a reaction the film caused. That became "only
  reactions count" in rule 2. Labelling my cold 20 later showed one more gap
  (a described scene followed by only an evaluation word), and I added a
  sentence to rule 1 for it.

**Other AI use, disclosed plainly**

- **Data formatting:** the AI turned my Word document of copied reviews into
  `labels.csv`. It fixed reviews that were merged or split by missing or extra
  blank lines, and removed one duplicate. I chose the review that replaced it.
- **Translation:** the AI made a Chinese–English side-by-side copy of the
  reviews for my own reading. It isn't part of the data, and I didn't use it for
  the cold 20.

**Pre-labelling disclosure:** The 20 rows marked `cold` in the `note` column
were labelled by me alone, with no AI. The other 180 were pre-labelled by AI
using my definitions and decision rules, with a one-line reason each and a ⚠️ on
the 63 it was unsure about (`prelabel_180.md`). I am reading every one and
keeping or correcting the label. Their `label` column in `labels.csv` stays
empty until I've checked them. Each finished row will be marked
`pre-labelled, checked` or `pre-labelled, corrected`.

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
