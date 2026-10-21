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

Overall accuracy on the held-out set will be at least 0.65.

**Why this target:** With 3 labels, random guessing gets about 0.33. Kaggle Discussions posts are fairly distinct in register (a finished technical write-up reads very differently from a platform rant), so I expect the model to clear "better than guessing" by a wide margin — but 200 posts total is a small dataset for a fine-tune, so I'm not setting this near ceiling. 0.65 is comfortably above the floor without assuming the model will be excellent.

---

## 2. Per-label performance (weakest label)

Every label will have an F1 of at least 0.55 on the held-out set, including whichever label ends up smallest.

**Why this target:** `complaint` and `open_question` are likely to be thinner than `technical_analysis` in a technical-heavy forum like Kaggle Discussions — Kaggle Discussions skews toward people sharing or asking about technical work. A smaller label means fewer training examples and a noisier test-split score (as few as 8 examples in test, per the notes above), so 0.55 is set lower than the overall-accuracy target on purpose, to be a realistic floor rather than a stretch goal.

---

## 3. Balance

No single label will make up more than 65% of the 200 labeled posts.

**Why this target:** The project's hard cap is 70%; I'm setting my own target 5 points under that as a safety margin, since `technical_analysis` is the label I'd expect to dominate by default if I only collect from the most visible/upvoted threads. Staying under 65% means I had to deliberately go find enough `complaint` and `open_question` posts rather than taking whatever came easiest.

> **Note added in unit 5, before any results were analysed (original criterion unchanged):** I ended with 161 labeled posts, not 200, because Kaggle limited how many posts I could collect (details in the README). The largest label is `open_question` at 55%, which is under the 65% target.

---

## 4. Consistency

When I label the same 20 posts twice — once "cold" in Milestone 4 and once again later, without looking at my first pass — I will agree with myself on at least 80% of them.

**Why this target:** My hardest boundary (`complaint` vs. `open_question`) is a judgment call about tone and intent, not a hard fact, so I don't expect 100% self-agreement. 80% means the decision rule I wrote is actually usable in practice, not just on the one borderline example I used to write it.

> **Note added in unit 5, before any results were analysed (original criterion unchanged):** I have no cold labels of my own. Claude labelled the 20 sample rows a second time from their text alone and matched its first pass on 20 of 20, but that is the same labeller twice, so it cannot test this criterion. As written, this criterion can't be measured. In unit 6 I expect to revise it, underneath this note, to use the agreement check against the staff labels.

---

## 5. Confidence

Posts the model predicts with high confidence (top probability above 0.80) will be correct at least 85% of the time.

**Why this target:** A classifier that's just as wrong when it's "sure" as when it's unsure isn't trustworthy to act on. 85% is above my overall-accuracy target (0.65) on purpose — confident predictions should be meaningfully more reliable than the average prediction, not just equal to it.

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
