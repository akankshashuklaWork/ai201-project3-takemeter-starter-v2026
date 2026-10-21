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

<!-- DRAFT — check this matches how you'd actually describe it, and personalize the wording before submitting. -->
TakeMeter sorts posts from **Kaggle Discussions**, the forum where Kaggle users talk about competitions, the platform, and machine learning in general. My classifier sorts each post into one of three labels: `technical_analysis` (a finished write-up reporting a concrete result from building or training something), `open_question` (any post that asks something and is still unresolved, even if it also vents or reports numbers), or `complaint` (a frustrated or cynical post about the platform or community that asks nothing).


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `technical_analysis`

**Definition:** A finished technical write-up that reports a concrete result from actually building or training something — real numbers, not just a claim.

**Example 1:**
> "Team Farmcore, 4th place of ~10,200 teams... The biggest gain came from fixing the labels, not the model: retrained on what the engine executed instead of what players ordered, plus one feeding rule, the same network went from 3 to 23 wins of 24 against our previous best."

**Example 2:**
> "0.959795: 5-fold OOF AUC, raw features only. 22: positions per row. 10.1M: trainable LoRA parameters on a frozen 0.6B backbone. ~52 min: per fold on one A100 (3 epochs, ~920 rows/s)."

### `open_question`

**Definition:** A post that asks something — technical or not — where the answer is still unresolved. If a post asks a question, or asks for help or for a fix, it is an `open_question`, even if it also vents or reports numbers. Neutral bug reports and requests for help count too.

**Example 1:**
> "I've noticed that the leaderboard medals have disappeared. Is this a bug, or were they intentionally removed? Will they be brought back?"

**Example 2:**
> "Someone who has 1M training examples per model will find more patterns than someone with 100K training examples per model. Is paying model providers for data generation going to be the key to winning?"

### `complaint`

**Definition:** A cynical or frustrated post about the platform or community that vents or demands broad change and asks nothing. If the post contains a question, it is an `open_question` instead.

**Example 1:**
> "I really agree, I've also seen people just copying other Grandmaster's work in not only this competition but the Google Deepmind and Playground series! This is just so annoying!"

**Example 2:**
> "This is a latent problem now-a-days. Firstly, compute costs + agents expenses is an issue and such notebooks accentuate the problem even further. Unfortunately Kaggle won't do anything regarding this - that is the biggest problem"

### The hardest boundary

**Which two labels:** `complaint` vs. `open_question`

**The decision rule I use every time:**
If the post asks something — a question, a request for help, or a request for a fix — it is `open_question`, even if it also vents or reports numbers. A post is a `complaint` only if it is frustrated or cynical about the platform or community and asks nothing. A `technical_analysis` post that ends by asking the community a question is also an `open_question`; a long write-up that only uses a rhetorical heading like "Why does this happen?" and then answers it stays `technical_analysis`.

**How I found this rule:** My first version of the rule was "a specific, answerable ask is `open_question`; venting about a pattern is `complaint`". I tested it by having Claude write 8 posts that sit on my boundaries, and I labelled them blind. I called every post that contained a question `open_question` and only the one with no question a `complaint`, including posts that were venting ("Why does Kaggle keep ignoring the notebook-copying problem?") and a post that reported its own numbers and then asked a question. So the rule I actually apply is "does it ask something?", and I rewrote the written rule to say that. I then re-checked the rows of `labels.csv` that contained a question mark: 10 rows that I had labelled `complaint` and 7 that I had labelled `technical_analysis` became `open_question`.

**The case that forced the first version:** a bug report — *"My RSNA Knee code competition submission has remained in Running status for over two hours... Could the Kaggle team check whether this submission is still being evaluated or is stuck?"* — is about a platform problem, like a complaint, but it is flat, factual and asks for one specific fix. That is an `open_question`.


---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public Kaggle Discussions (kaggle.com/discussions), read in October 2026. I used first posts and comments from busy competition threads (RSNA Knee Abnormality Detection, Name That LLM, Kaggriculture, ARC Prize, Gemma 4 Developer Agent, a Playground series), plus the General, Getting Started, Questions & Answers and Product Feedback forums. Competition write-ups supplied most of the `technical_analysis` rows. Nothing was behind a login. Each row's source URL is in `data/kaggle_sources.csv` (the `row` column is the line number in `labels.csv`).

I skipped posts that fit none of my three labels (praise like "great write-up!", welcome messages, staff announcements, jokes, arguments between two users), posts in other languages (my model is English-only) and the very long Seaborn tutorials. So the file is not a random sample of Kaggle Discussions. It is a sample of posts that fit my labels, and that is a limit on what the classifier can claim.

**How I labelled them:** Claude collected the posts and pre-labelled all 161 using my label definitions and my complaint-vs-open_question decision rule. After my blind stress test I changed the rule to "does the post ask something?", and Claude re-labelled the 17 rows that contained a question. Claude then re-read all 161 rows in a second pass against the final rule and found nothing else to change. The `note` column of every row says this.

**No cold labels:** The brief asks for 20 rows labelled cold, by me, with no help. I did not do that. Instead Claude labelled the 20 rows from `cold_20_to_label.csv` (chosen at random with a fixed seed) again from their text alone and matched its first pass on 20 of 20. That number means little, because it is the same labeller twice and Claude had labelled those rows before. Those rows are marked in `note` as NOT cold labels by me. I also did not read and correct the rows myself, so the dataset has no human-checked rows. <!-- TODO (Akanksha): if you do read the rows or label any cold, edit this paragraph and the `note` column to say so. -->

I did not reach the 200 rows the brief asks for. I got 161, because Kaggle rate-limited my collection (it refused requests after I fetched too quickly) and because many posts fit none of my labels. Under the brief's stop rule that is above the 150-row floor.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `open_question` | 88 | 55% |
| `technical_analysis` | 38 | 24% |
| `complaint` | 35 | 22% |
| **Total** | 161 | 100% |

No label is above 70% (the largest is 55%). `complaint` and `technical_analysis` are on the thin side. `open_question` is large partly because I count every post that asks something as one.

**Three hard cases**

**1.**
> *The post:* "I don't understand, how suddenly kaggle tpu q have been going this big lately." (Product Feedback)
>
> *Could have been:* `open_question` or `complaint`
>
> *I chose:* `complaint`, because it names a problem affecting many users (the TPU queue) and asks for nothing specific. Under my decision rule an `open_question` asks for a concrete check or fix, and this one only expresses frustration and puzzlement.

**2.**
> *The post:* "@macruzbar The Code section is now flooded with low-effort ensemble notebooks that offer little to no originality, making it increasingly difficult to find genuinely valuable notebooks. Could Kaggle give users the ability to downvote this kind of low-quality content?"
>
> *Could have been:* `open_question` or `complaint`
>
> *I chose:* `open_question`, because it ends by asking something. My first pass called it a `complaint` because most of the post vents about a pattern, but my final rule is that a post that asks something is an `open_question` even if it also vents.

**3.**
> *The post:* "Really clear write-up. Since the random-init backbone matched pretrained Qwen (0.960436 vs 0.960444), it seems the gain comes from the architecture and not from language pretraining. Did you try removing the causal mask so CLS-style attention sees all features bidirectionally?..."
>
> *Could have been:* `technical_analysis` or `open_question`
>
> *I chose:* `open_question`, because the post reports no result of its own. It quotes someone else's numbers and ends by asking a question that is still unresolved.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, seed 42 (the notebook defaults).

**Anything I changed from the defaults, and why:** Nothing. I changed only `LABELS` (to my three labels) and `NAME`.

**What I trained on:** Apple Silicon GPU (`mps`), torch 2.14.1. The run took about eight seconds. The device is recorded in `results.json`.

**Split sizes:** 112 train / 24 validation / 25 test (70/15/15, stratified).

| Label | train | val | test |
|---|---|---|---|
| `technical_analysis` | 26 | 6 | 6 |
| `open_question` | 61 | 13 | 14 |
| `complaint` | 25 | 5 | 5 |

`technical_analysis` has 6 and `complaint` has only 5 test examples (under the 8 the brief flags), so their scores will move a lot between seeds.

I committed `results.json` and `test_split.csv` and have not analysed them yet. That is next unit's work. One thing I could not miss when the notebook printed its summary: the model put almost every test post into `open_question`, the biggest label.

---

## How I Used AI

**Moment 1: writing my acceptance criteria.**

- *What I asked for:* I asked Claude to write my five acceptance criteria for me.
- *What came back:* Five criteria with targets and short reasons (accuracy at least 0.65, every label's F1 at least 0.55, no label above 65%, 80% self-agreement on my 20 cold posts, and 85% accuracy on confident predictions).
- *What I changed:* I did not change any of the five targets or reasons. They stand as Claude drafted them and as I filed them before training. I also asked Claude for one reason each target might turn out too easy and one reason it might turn out too hard, and I made no number changes from that. Two criteria no longer fit my final data (criterion 3 says 200 posts and I have 161; criterion 4 needs cold labels that I don't have). I recorded that in `criteria.md` as notes under each criterion, without editing the original lines.

**Moment 2: collecting and pre-labelling the posts.**

- *What I asked for:* I asked Claude to collect public Kaggle Discussions posts, aiming for about 70 per label, and to pre-label them using my definitions and decision rule.
- *What came back:* 161 posts from Kaggle's discussion pages, each with a label. Claude also noticed that one of my `open_question` examples was not a question and swapped in a real one.
- *What I changed:* Nothing by hand. Claude re-labelled 17 rows after my stress test changed the decision rule, and then re-checked all 161 rows.

**Moment 3 (extra): stress-testing my labels.**

- *What I asked for:* I asked Claude to write 8 posts that sit on the boundaries between my labels, and I classified them blind.
- *What came back:* 8 made-up posts (not real Kaggle posts). I called all of them `open_question` except the one with no question in it, and Claude's own read split from mine on 4 of 8.
- *What I changed:* I replaced my written decision rule with the rule I actually apply ("if the post asks something, it's `open_question`") and re-labelled the 17 affected rows in `labels.csv`. I did not adopt the two tie-breakers Claude suggested.

**Pre-labelling disclosure:** All 161 rows were pre-labelled by Claude, and Claude also did the second-pass review. I did not label any rows cold myself and did not review the rows row by row; see "No cold labels" in The Dataset.

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
