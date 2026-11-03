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
   A reason counts only if the reason itself points to something in the film. Another evaluation, a comparison that doesn't say what's alike, or a claim about the producers or the audience doesn't count.
   - "Good Actors, Good Director" → only evaluation words → `hot_take`
   - "Bad CGI, no sense of heights" → "no sense of heights" says what the CGI fails to do → `critique`
   - "A simple plot like this did not need a 2.5h movie" → a checkable fact → `critique`

2. **`experience` vs `hot_take`: only reactions count.** 
   A reaction the film caused counts (crying, laughing, falling asleep, walking out, turning it
   off). Where, how or with whom they watched (on a plane, at home, with my wife) is context and does not count. A lasting change in the writer caused by the film ("I have learned to be happy with what I have") also counts as a reaction. If a review has both a reaction and a verdict, I pick whichever takes up more of it.

3. **Order: rule 1 first.** 
   If it finds anything specific, the label is `critique` and I stop. 
   Only if it doesn't do I use rule 2.

Complaints about the rating or the hype ("Overrated", "Really? 10/10?") follow the same rules. I considered a fourth label for them: about 60 of my 200 reviews comment on the rating. It wouldn't hold, because a rating complaint is a topic, not a way of writing. "Overrated … How the wardens didn't notice he was chipping through the wall everyday to escape is beyond me" argues from a plot detail, while "Dont get why is this greatest here, lol ok" argues from nothing. They needed to be `critique` and `hot_take`, not one shared label.

**How the rule was tested:** 
I classified eight boundary reviews an AI wrote for me. I got 3 of 8 right on the first pass. The misses showed two leaks. I was counting a name as specific ("Ryan Gosling is great"), which led to the blank-out-the-names step. A single side remark about the viewing conditions ("watched this on a plane with terrible headphones") was turning a review into `experience`, which led to "only reactions count".

Labelling the real 200 found three more gaps, and each became a sentence above. The first was a scene described with only an evaluation word ("It was touching when the multilingual said goodbye to the solar system"). The second was a reason that is itself only an evaluation or an outside comparison ("as designed and engineered as N-SYNC", "another 'Alice' situation"). The third was a lasting change rather than an in-the-moment reaction ("I have learned to be happy with what I have").

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

> **Status:** At the unit 5 deadline, collection was done and only the 20 cold
> rows were labelled. I finished labelling all 200 afterwards, then ran the
> training (Milestone 5, see The Training Run). The worksheets are committed:
> `cold_20.md` holds my cold labels and re-check, `prelabel_180.md` holds the
> AI's pre-labels with my check on each, and `disagreements_20.md` holds the
> 20 where I disagreed with the AI and how each was settled.

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
definitions and rules. I read every one, with the AI's Chinese translation
beside the English to go faster, and went back to the English whenever a
review was close. I agreed with 160. I disagreed with 20, and for those I
compared my reason against my own rules and my earlier labels. 5 ended up
changed from the AI's label (`pre-labelled, corrected` in the `note` column).
For the other 15, re-checking showed my objection was counting an evaluation
word or a bare comparison as specific, so the AI's label stayed
(`pre-labelled, checked`).

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `critique` | 94 | 47% |
| `hot_take` | 95 | 47.5% |
| `experience` | 11 | 5.5% |
| **Total** | 200 | 100% |

No label is above the 70% cap, and `critique` and `hot_take` are close to
even. They're also spread evenly across films: each film has between 6 and 13
of each, so the model can't learn a label from the film's name. `experience`
is the problem. At 11 rows it misses my own criterion 4 (at least 30), and the
test split will hold only one or two, so its F1 next unit will be close to
meaningless. I'm keeping it as a third label rather than merging it into
`hot_take`, and reporting the miss instead of hiding it. The reason it's thin
is a finding about IMDb reviews: almost everyone writes to judge the film, and
under my rules a review counts as `experience` only if the reaction is the
main thing and nothing in it is specific. Collecting more `experience` reviews
is the obvious fix to try in unit 6.

**Hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

Five, not three. Each one either changed a rule or tested one. Numbers are row
numbers in `labels.csv`.

**1. A list of evaluation words (#11, Obsession, one of my cold 20)**
> *The post:* "OVERRATED. This movie isn't bad, I love horror and saw this in theaters asap. It's a decent movie but to call this one of the greatest horror movies in recent time is insane. People saying that this movie left them jaw dropped and that it's soooo good is a crazy take. This movie has a great Jumpscare, a decent plot, okay acting with the main character having great acting, the ending was decent, overall just a decent movie I'd rate it no higher than a 6.5 if you think this is a 10 then idk."
>
> *Could have been:* `critique` or `hot_take`
>
> *I chose, because:* `hot_take`. Cold, I called it `critique` because it names five parts of the film. When I blanked out the names, every part was followed only by an evaluation word: great, decent, okay, great, decent. Naming more parts doesn't make a review more specific. This was one of the six cold labels that changed on re-check, and it's why my cold 20 started at 80% `critique`.

**2. A lasting change, not an in-the-moment reaction (#144, Life Is Beautiful)**
> *The post:* "Life is really beautiful. Just yesterday I saw the movie, basically am not a happy person. After I saw this movie I have learned to be happy with what I have. I saw his Oscar award videos before I saw the move. Then I understood. Thanks Robert"
>
> *Could have been:* `experience` or `hot_take`
>
> *I chose, because:* `experience`. Every reaction listed in rule 2 happens while watching: crying, laughing, falling asleep, walking out. This one is a change that lasted after the film: "I have learned to be happy with what I have." Nothing in it is about the film itself, and the change is the whole point of the review. I decided a lasting change caused by the film counts as a reaction and added that sentence to rule 2.

**3. A reason that is only a comparison (#93, Spider-Man)**
> *The post:* "Wait for it to come out on video. First the good: Great effects, good fight scenes, fantastic acting, direction, and cinematography. Now the bad: Formula, formula, formula. This movie was as designed and engineered as N-SYNC. The producers didn't even try to disguise their quest for money with an interesting plot. And the ugly: The smarmy love-story. It made me want to wretch. How the actors kept straight faces while reciting some of the corniest and sappiest lines I've ever heard is worthy of an Academy Award. I haven't seen anything so maudlin since the Sammy Maudlin show."
>
> *Could have been:* `critique` or `hot_take`
>
> *I chose, because:* `hot_take`. I first read "as designed and engineered as N-SYNC" as an argument, because it names a clear comparison. But N-SYNC is a boy band, and the comparison only says "formulaic" in another way. It never says which parts of the film are formula. "The producers didn't even try" is about the producers, not the film. "Smarmy", "corniest" and "sappiest" are evaluation words. This is where I added to rule 1 that a reason has to point to something in the film itself. "It made me want to wretch" is a reaction, but most of the review is verdict, so rule 2 still gives `hot_take`.

**4. Short, but specific (#76, Project Hail Mary)**
> *The post:* "Could have done better. I had not read the book so I went in with an open mind. I found the movie too long and a bit silly. They were seemingly trying to convey the complexity of the human experience but it just came up short. As with many films today they just had too many endings and it dragged along. The score was mismatched, loud and grating. The could have done better."
>
> *Could have been:* `critique` or `hot_take`
>
> *I chose, because:* `critique`. Most of it is evaluation ("too long", "a bit silly", "came up short"). I hesitated because the reasons are so brief. But "too many endings" is something you could check by watching: the film has several closing sequences. "Loud" says what the score is like, not just that it's bad. Rule 1 says one specific sentence is enough, so length doesn't decide it. The AI had pre-labelled this one `hot_take`, and I changed it.

**5. Reaction or verdict, roughly half and half (#3, Obsession)**
> *The post:* "Best horror I've seen in a long time. This movie had so much range, you'd be scared shitless and then laugh the next second. Also I had my biggest jump scare ever which I never really fall for, but I literally flew back in my seat and rocked the whole chair lmao. Major respect to the girl playing Nikki she was phenomenal, and props to the crew and actors for making such a quality movie with a low budget. This really is the best movie I've seen in the past 3 years and I'm usually reluctant to ever say something like that but I'm so impressed, this is a must see!"
>
> *Could have been:* `experience` or `hot_take`
>
> *I chose, because:* `experience`. Nothing passes rule 1: "phenomenal" and "quality movie" are evaluation words. So it comes down to rule 2's "whichever takes up more of it". Two sentences are reactions: being scared and then laughing, and flying back in the seat and rocking the chair. Three are verdict. That's close to even. I chose `experience` because the jump scare is the only concrete thing that happens in the review, and the praise around it reads as the result of that reaction. The AI had said `hot_take`. This is the closest call in my data, and it shows "whichever takes up more" is hard to apply when the two halves are about the same length.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, seed 42, `MAX_LENGTH` 256. Trained on `mps` (torch 2.2.2). The notebook calls it "Apple Silicon GPU", but my Mac is an Intel Mac, so this is its AMD GPU running through Apple's Metal. Training took about 3 minutes.

**Anything I changed from the defaults, and why:**

*`MAX_LENGTH` 128 → 256.* I counted tokens in all 200 reviews: the median is 95 and the longest is 229. At 128, 42 reviews would be cut off, and 29 of them are `critique`. A critique often gives its specific reason after an opening verdict ("Great movie! … the ending works because …"), so cutting the end can leave only the verdict and teach the model that a `critique` looks like a `hot_take`. At 256 no review is cut. The cost is slower training, which doesn't matter at 139 training rows.

*Environment.* My machine is an Intel Mac. The newest torch with an Intel Mac wheel is 2.2.2, and it only works with NumPy 1.x, but `pip install -r requirements.txt` gave me NumPy 2.5.4. Training failed with `RuntimeError: Numpy is not available`. I changed three package versions inside `.venv` and did not edit `requirements.txt`:

- `numpy<2` (got 1.26.4), so torch 2.2.2 can use it.
- `scipy<1.14` (got 1.13.1), because SciPy 1.18.1 needs NumPy 2.
- `pyarrow==22.0.0`, because pyarrow 26.0.0 (used by `datasets`) needs NumPy 2 and failed with `ImportError: pyarrow requires NumPy 2.0 or newer, found 1.26.4`. I tested 22.0.0 with NumPy 1.26.4 before installing it.

None of these change how the model trains. They only make the libraries load.

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->

| | train | val | test |
|---|---|---|---|
| critique | 66 | 14 | 14 |
| hot_take | 66 | 15 | 14 |
| experience | 7 | 2 | 2 |
| **total** | **139** | **31** | **30** |

`experience` has only **2** reviews in the test split, far fewer than 8. Its F1 can only be 0, about 0.5 or 1.0, so it will jump between seeds. The model also only had 7 `experience` reviews to learn from. On the seed-42 run it got `experience` F1 0.000: it called both test `experience` reviews `hot_take`.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

I used Claude (Claude Code) as a tutor throughout. These notes cover all of
unit 5.

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

- **Training setup (Milestone 5):** the AI found why the libraries wouldn't
  load (torch 2.2.2 on my Intel Mac needs NumPy 1.x, and SciPy and pyarrow
  needed NumPy 2). It tested pyarrow 22.0.0 against NumPy 1.26.4 in a separate
  folder before I installed it. I ran the installs. It also counted the tokens
  in my 200 reviews and recommended `MAX_LENGTH` 256. I accepted that. I ran
  the notebook myself.
- **Data formatting:** the AI turned my Word document of copied reviews into
  `labels.csv`. It fixed reviews that were merged or split by missing or extra
  blank lines, and removed one duplicate. I chose the review that replaced it.
- **Translation:** the AI made a Chinese–English side-by-side copy of the
  reviews for improving my own reading efficiency. It isn't part of the data. 
  I didn't use it for the cold 20. I did read it beside the English when checking the other 180.
- **Three-seed run (unit 6, Milestone 2):** I ran notebook sections 1–7
  myself. The AI read the per-seed confusion matrices from
  `results_three_seeds_before.json`, filled in the run log numbers (I left
  the Verdict column for Milestone 4), and drafted the biggest-confusion
  paragraph. I checked it against the matrices and kept it.
- **Staff set (unit 6, Milestone 3):** the AI made a Chinese–English copy of
  `data/staff_taxonomy.md` and of the 30 posts, in a shuffled order with no
  label hints. I chose all 30 labels myself and wrote down which rule I used
  for each. The AI copied my labels from that file into `my_staff_labels.csv`
  and converted the staff labels from `.xlsx` to `data/staff_labels.csv`
  without reading them. After `agreement.py` reported 30/30, it compared my
  rule notes with the staff `why` column and drafted the Agreement Report.

**Pre-labelling disclosure:** The 20 rows marked `cold` in the `note` column
were labelled by me alone, with no AI. The other 180 were pre-labelled by AI
using my definitions and decision rules, with a one-line reason each and a ⚠️ on
the 63 it was unsure about (`prelabel_180.md`). I read every one and marked
it agree or disagree. I disagreed on 20. I settled each of those with the AI
by testing my reason against my own rules (`disagreements_20.md`). On 6 I
was undecided and took the AI's reading once it showed that my reason was
itself only an evaluation or a bare comparison. Final result: 175 kept as
pre-labelled (`pre-labelled, checked`) and 5 changed (`pre-labelled,
corrected`).

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.433 | 0.567 | +0.133 |
| Macro F1 | 0.287 | 0.388 | +0.101 |
| F1 — `critique` | 0.348 | 0.645 | +0.297 |
| F1 — `experience` | 0.000 | 0.000 | +0.000 |
| F1 — `hot_take` | 0.514 | 0.519 | +0.004 |

Both columns are on the same 30 held-out reviews (`test_split.csv`, seed 42).
The baseline is `facebook/bart-large-mnli` zero-shot, prompted with my
definitions from `label_definitions.txt` (`baseline_results.json`).

**What my definitions were worth:** the same baseline with bare label names
only (`baseline_bare_names_results.json`):

| Measure | Bare names | With my definitions | Difference |
|---|---|---|---|
| Overall accuracy | 0.233 | 0.433 | +0.200 |
| Macro F1 | 0.219 | 0.287 | +0.068 |
| F1 — `critique` | 0.333 | 0.348 | +0.015 |
| F1 — `experience` | 0.190 | 0.000 | −0.190 |
| F1 — `hot_take` | 0.133 | 0.514 | +0.381 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

*Written after seeing my trained results from unit 5, but before running the
baseline.*

- I predict the trained model will beat the baseline on overall accuracy by
  about 10 points, because the baseline has never seen a single labelled
  review and will probably land near the 0.47 you'd get by always guessing
  `critique`.
- I expect the biggest gap on the `critique` vs `hot_take` boundary: almost
  every review names an actor or the plot somewhere, so a zero-shot model
  reading my definitions will likely call most reviews `critique`, while the
  trained model learned from 200 examples that naming something isn't enough
  unless the review says what it is like or why.
- On `experience` the baseline might actually win, because my definition lists
  concrete reactions like crying, falling asleep or walking out that a general
  model can match directly, while my trained model saw too few `experience`
  examples and scored 0 F1 — though with only 2 `experience` reviews in the
  test split, neither number will mean much.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

Fine-tuning added something real, but almost all of it is on critique. The +0.13 accuracy gap is 4 more reviews right out of 30, and it comes from critique F1 rising from 0.35 to 0.65. The baseline caught only 4 of 14 critique reviews and called 21 of 30 reviews hot_take. My prediction had that direction backwards. I expected the baseline to over-call critique because most reviews name an actor or the plot. Instead, it seems to read any judgement as a verdict, and it can’t check the second half of my definition (“says what that thing is like or why”). That second half is what 200 labelled examples taught the trained model.

On hot_take, fine-tuning added nothing (0.519 vs 0.514). On this split, my trained model is no better at it than a general model reading a one-line definition. On experience, neither model got either of the 2 test reviews, so this split leaves that label untested.

My definitions were worth 0.20 accuracy on their own (0.233 with bare names, 0.433 with definitions). With bare names, the baseline was worse than chance, mostly because it called 19 of 30 reviews experience. All of this comes from one seed and 30 reviews, so Milestone 2’s three seeds will show whether the gap holds.

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.567 | 0.833 | 0.500 |  |
| 2. Per-label F1 | `critique` ≥ 0.60, `hot_take` ≥ 0.60, `experience` ≥ 0.40 | c 0.645 · h 0.519 · e 0.000 | c 0.897 · h 0.828 · e 0.000 | c 0.133 · h 0.651 · e 0.000 |  |
| 3. Confidence (most-sure third minus least-sure third) | ≥ +0.20 | 0.500 − 0.600 = −0.10 | 1.000 − 0.700 = +0.30 | 0.600 − 0.300 = +0.30 |  |
| 4. Balance (`labels.csv`) | biggest label ≤ 55%, `experience` ≥ 30 rows | 47.5% · 11 rows | 47.5% · 11 rows | 47.5% · 11 rows |  |
| 5. Agreement with the staff set | ≥ 0.75 (23 of 30) | pending Milestone 3 | — | — |  |

Criterion 4 is about the labelled file, not a split, so it is the same on every
seed. Criterion 5 is one comparison against the staff set, not a per-seed
number.

All three seeds ran on `mps` (Apple Silicon GPU), with the unit 5 settings:
`distilbert-base-uncased`, 3 epochs, learning rate 2e-5, batch 16,
`MAX_LENGTH` 256 (`results_three_seeds_before.json`).

**Spread across seeds** (notebook section 6):

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.567 | 0.833 | 0.500 | 0.333 |
| Macro F1 | 0.388 | 0.575 | 0.261 | 0.313 |
| F1 — `critique` | 0.645 | 0.897 | 0.133 | 0.763 |
| F1 — `hot_take` | 0.519 | 0.828 | 0.651 | 0.309 |
| F1 — `experience` | 0.000 | 0.000 | 0.000 | 0.000 |

Accuracy moved 0.333 across three seeds, more than three times the 10-point
line for a stable measure. With 30 test reviews, one seed's score depends
heavily on which reviews landed in the test split.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42 (notebook section 7):

| true \ predicted | critique | hot_take | experience |
|---|---|---|---|
| **critique** | 10 | 4 | 0 |
| **hot_take** | 7 | 7 | 0 |
| **experience** | 0 | 2 | 0 |

Seed 7, from `results_three_seeds_before.json`:

| true \ predicted | critique | hot_take | experience |
|---|---|---|---|
| **critique** | 13 | 1 | 0 |
| **hot_take** | 2 | 12 | 0 |
| **experience** | 0 | 2 | 0 |

Seed 2024, from `results_three_seeds_before.json`:

| true \ predicted | critique | hot_take | experience |
|---|---|---|---|
| **critique** | 1 | 13 | 0 |
| **hot_take** | 0 | 14 | 0 |
| **experience** | 0 | 2 | 0 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

On seed 42, 7 real `hot_take` reviews were called `critique` and 4 went the
other way. Taken alone, that looks like a boundary the model finds hard in
both directions. Across all three seeds, though, the biggest off-diagonal
number is seed 2024's 13: 13 of 14 real `critique` reviews were called
`hot_take`, and on that split the model predicted `hot_take` for 29 of 30
reviews. So the model's main error is not a steady lean in one direction. It
sits close to 50/50 between `critique` and `hot_take`, and whichever split it
gets tips it one way or the other. The one direction that holds on every seed
is `experience`: all 2 `experience` reviews went to `hot_take` on all three
seeds, and the model never predicted `experience` once.
---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 on all three seeds | MISSED | Only seed 7 cleared it (0.833). Seed 42 (0.567) and seed 2024 (0.500) did not. |
| 2 | Per-label F1 | `critique` ≥ 0.60, `hot_take` ≥ 0.60, `experience` ≥ 0.40 on all three seeds | MISSED | `experience` was 0.000 on every seed. `hot_take` also missed on seed 42 (0.519), and `critique` on seed 2024 (0.133). Only seed 7 met both of the big labels. |
| 3 | Confidence | most-sure third ≥ 0.20 more accurate than least-sure third, on all three seeds | MISSED | Seeds 7 and 2024 gave +0.30, but seed 42 gave −0.10: the confident third did worse than the unsure third. |
| 4 | Balance | biggest label ≤ 55% and `experience` ≥ 30 rows | MISSED | The first half holds (`hot_take` is 95 of 200, 47.5%). The second half doesn't: `experience` has 11 rows. |
| 5 | Agreement with the staff set | ≥ 0.75 (23 of 30) | MET | 30 of 30. |

I am not revising any criterion. All five could be measured and gave a clear
verdict. The `experience` F1 in criterion 2 is computed on only 2 test reviews,
which makes it noisy, but it is still a number, and it was 0 because the model
never predicted `experience` at all. Loosening it would be lowering a target I
missed, not fixing a broken one.

**Diagnoses**

The four misses come from two causes. Criteria 2 (`experience`) and 4 share
one; criteria 1, 3 and the `critique`/`hot_take` part of 2 share the other.

*Cause A: too few `experience` examples (criterion 4, and `experience` in
criterion 2).* `labels.csv` has 11 `experience` rows. That leaves about 8 for
training and 2 for each test split. The model never predicted `experience` on
any seed: in all three confusion matrices both `experience` reviews went to
`hot_take`. This is a data problem, not a model problem, and it is one I
flagged when I wrote criterion 4. Under my rule, `experience` needs a reaction
*and* nothing specific, and most reviews that describe a reaction also name
something in the film, which makes them `critique`. The baseline also scored
0 on `experience` with my definitions, so the definition alone can't make up
for the missing examples.

*Cause B: the model barely learned the `critique`/`hot_take` boundary, so the
split decides the score (criterion 1, criterion 3, and the `critique` and
`hot_take` parts of criterion 2).*

- **The spread is too wide to be a measurement.** Accuracy moved 0.333 across
  seeds. Seed 7 got 25 of 30 right; seed 2024 called 29 of 30 reviews
  `hot_take`.
- **The model is close to guessing the label mix.** Final eval loss was 0.87–0.89
  on every seed, and the loss of a model that only knows the label proportions
  in `labels.csv` is about 0.87. The model's average confidence was about 0.47
  whether it was right or wrong (0.467 vs 0.468 on seed 42). A model this
  unsure tips one way or the other on small differences in the training split,
  which matches the matrices: seed 42 leans `critique`, seed 2024 leans
  `hot_take`.
- **Criterion 3 fails for the same reason.** When every confidence is about
  0.47, sorting into thirds sorts on noise. Each third is 10 reviews, so one
  review moves a third's accuracy by 0.10. Even the +0.30 on seeds 7 and 2024
  sits on a right-vs-wrong confidence difference of only 0.01, so I don't read
  those two as the model's confidence meaning something.
- **It is probably not my labelling.** My agreement check was 30 of 30 under
  the staff rules, including all 7 posts staff marked borderline. That is the
  only evidence I have that separates a labelling problem from a model
  problem, and it points away from "I applied the boundary two ways." It
  doesn't fully rule it out: the staff rules test an explicit fact, which is
  easier than my "says what that thing is like or why", and 180 of my 200
  were AI pre-labels I checked rather than labelled cold.
- **The boundary is also genuinely hard.** The zero-shot baseline failed on
  the same pair, calling 21 of 30 reviews `hot_take`. But seed 7 reached
  `critique` F1 0.897 and `hot_take` F1 0.828, so the boundary can be learned
  from these labels on some splits.
- **What I think the cause is.** With 139 training reviews and a batch size
  of 16, 3 epochs is only about 27 training steps at learning rate 2e-5. That
  looks like too little training to move the model far from the label
  proportions. This is the one place my diagnosis points at a training
  setting, not at the labels.

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

**Agreement rate:** 30 / 30 = 100% (`agreement_results.json`)

I labelled all 30 in `my_staff_labels.csv` under `data/staff_taxonomy.md`
before I opened `data/staff_labels.csv`. I read the posts in a shuffled order,
with a Chinese translation beside the English, and wrote down which rule I used
for each one.

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

None. I matched staff on every post, so there is nothing to adjudicate and
nothing to sort into piles (a), (b) or (c).

**Was the set easy, or is the rule tight?** Some of both.

- *Part of it is easy.* In `staff_posts.csv` the posts are grouped by label in
  blocks of ten, and staff marked only 7 of the 30 as borderline. The other 23
  are textbook cases, such as "Heart rate zones are astrology for runners."
- *The hard ones still matched.* I agreed on all 7 borderline posts. These
  include `sp_014`, the canonical rule-1 case: the 3% is decoration on a
  conclusion already reached. They also include `sp_005` and `sp_010`, where
  a hedged or half-hot-take post still rests on a checkable number. I read the
  staff rules as tight because each one asks a single test question: does the
  fact carry the argument, is it about the writer or the world, is there a
  load-bearing fact.
- *Same labels, not always the same reasoning.* On two borderline posts, the
  rule I noted was not the one staff used. On `sp_019` ("Nobody actually
  enjoys the long run…") I used rule 1 (no fact, so not `analysis`). Staff
  used rule 2 (a general claim, not the writer's own experience). On `sp_028`
  ("Third DNF this year…") I used rule 2. Staff used rule 3 (a countable fact
  that nothing is argued from). Both pairs land on the same label, but a
  100% rate hides that we got there by different routes.

**What the pattern in my disagreements tells me:**

With no disagreements, this check is evidence that I can apply a written
boundary rule consistently. It makes "I labelled my own 200 inconsistently"
less likely as the main cause of my model's `critique`/`hot_take` confusion,
which points the diagnosis more toward the data and the training. It doesn't
rule it out, though. The staff rules turn on explicit, checkable facts (a
number, a date). My own `critique` rule turns on whether a review "says what
that thing is like or why", which is a softer test. Also, 180 of my 200 labels
were AI pre-labels that I checked rather than labelled cold. Getting 30 of 30
here is not proof that those 200 are as consistent.

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
