# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a criterion.

Under each, write a sentence or two on **why that number**. A reason that says something about your data or your taxonomy earns credit — *"I picked 0.60 F1 for `reaction` because it's my smallest label and I only have about 50
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

Accuracy on the held-out test split is at least **0.65** on all three seeds.

**Why this target:** Three labels make a coin-flip guesser about 0.33, but that isn't my real floor. I expect `critique` to be roughly half of my data, so a model that always guesses `critique` already scores about 0.50. With a test split of 30 reviews, 0.65 means about 20 right, roughly five reviews better than that do-nothing guesser. I didn't go higher because my own first pass on the stress test was 3 of 8. If the boundary is that hard for me, the model learning from my labels won't be near-perfect either.

---

## 2. Per-label F1

On all three seeds, the F1 for `critique` and for `hot_take` is each at least **0.60**, and the F1 for `experience` is at least **0.40**.

**Why this target:** `critique` vs `hot_take` is my hardest boundary. If the model has learned the blank-out-the-names rule at all, both sides should clear 0.60. If it hasn't, one side collapses and this catches it even when accuracy looks fine. `experience` gets a looser target because it is my smallest label. I expect about 5 of them in a 30-review test split, so finding one more or one fewer moves its recall by 0.20. A 0.60 target there would be measuring luck. 0.40 still rules out a model that never predicts `experience` (F1 of 0).

---

## 3. Confidence

On all three seeds, accuracy on the model's most confident third of test predictions is at least **0.20** higher than on its least confident third.

**Why this target:** Each third is only about 10 reviews, so accuracy there moves in steps of 0.10. A gap of 0.10 is one review and could easily be noise. 0.20 means at least two more right out of ten. That is the smallest gap I'd believe says the model's confidence means something, for example that a confident `critique` call can be trusted more than a hesitant one.

---

## 4. Balance

In `labels.csv`, no label is more than **55%** of the 200 rows, and `experience` has at least **30** rows.

**Why this target:** The brief's cap is 70%, but at 70% the always-guess-the- biggest-label floor would be 0.70, above my own accuracy target in criterion 1, which would make criterion 1 meaningless. 55% keeps that floor well below 0.65. 30 `experience` rows gives about 4 or 5 in the test split. That's still thin, but it's the least I'd accept for criterion 2's `experience` F1 to be computed on more than a handful of reviews. I'm not sure I'll reach it: under my rules, `experience` needs a reaction and nothing specific, and many reviews that mention a reaction also name something specific.

---

## 5. Labelling consistency

On the 30-post staff set, my labels agree with the staff labels on at least **0.75** of posts (23 of 30).

**Why this target:** The staff taxonomy (`analysis`, `hot_take`, `reaction`) draws almost the same boundaries as mine: evidence vs. none, and a reaction vs. a verdict. So I should be able to apply it fairly well. I didn't set it higher because applying someone else's rule is harder than applying my own, and my 3-of-8 first pass on my own stress test shows how easily I read a name or an evaluation word as evidence. Below 23 of 30 I'd suspect my own 200 labels are inconsistent too.

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
