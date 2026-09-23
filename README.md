# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

## Chunking Strategy

**Chunk size:** 700 characters
**Overlap:** n/a (see below — my chunker doesn't cut mid-post, so there's nothing to overlap)

<!-- What about YOUR documents made you pick these numbers? -->

When I ran `python app.py index` with the starter chunker, it printed 88 documents in and 88 chunks out — the 800-character window never actually cut anything, because when I read the corpus in Milestone 1 I saw every post in `campus_life` is short (179–563 characters). Each post is also about exactly one thing: one dorm, one course, one dining hall. The paragraphs inside a post ("the good", "the bad", laundry costs, noise) are different details about that same thing, not different topics.

That told me the right move wasn't to pick a smaller chunk size and start cutting posts up — a chunk like "Laundry costs $2.00 wash, $1.75 dry, app-based" on its own doesn't say which building it's about, so splitting mid-post would actually make chunks worse, not more focused. Instead I wrote my own chunker (`chunker.py::split_documents`) that splits on paragraph breaks and glues paragraphs back together up to `CHUNK_SIZE`, only cutting between paragraphs, never mid-sentence. I set `CHUNK_SIZE` to 700 — a bit above the longest post in the corpus (563 characters) — so in practice every post still comes out as one whole chunk, but now that's a decision I made on purpose after reading the documents, not an accident of a generic 800-character default.

## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160.txt#0` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_hist_118_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_pellew_dining_hall_followup.txt#0` — produced by: `chunker.py::split_documents`

```
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt#0` — produced by: `chunker.py::split_documents`

```
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

Reading these back: each one stands on its own — you could answer "when's the add/drop deadline" or "what's Innisfree Hall like" from the chunk alone, without needing the chunk before or after it. That's the payoff of keeping a post whole instead of cutting it at a fixed character count.

## Sample Answer

**Question:** How many pass/fail options can you use throughout your degree?

**Answer:**

```
You can use a maximum of eight pass/fail options across a degree (and two per year).

Source: admin_pass_fail_option.txt
```

**My relevance cutoff:** 0.6 (the starter default  I checked it against my own numbers instead of just keeping it because it was already there)

I ran my five questions from `questions.py` through `python app.py retrieve` and wrote down the best distance for each, then did the same for the five `OUT_OF_SCOPE` questions:

| Question | In corpus? | Best distance |
|---|---|---|
| How many pages does the printing quota cover? | yes | 0.418 |
| How many pass/fail options can you use throughout your degree? | yes | 0.241 |
| Does the registrar or the department decide if transfer credits count toward the major? | yes | 0.571 |
| Is street parking on Verrill legal? | yes | 0.360 |
| Does campus offer free bike registration? | yes | 0.621 |
| What is the capital of Mongolia? | no | 0.825 |
| How do I change the oil in a diesel engine? | no | 0.934 |
| Who won the 1994 World Cup? | no | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | no | 0.844 |
| How do I write a for loop in Rust? | no | 0.896 |

The in-corpus group (ignoring the two rows below) sits at 0.24-0.42, and every out-of-scope question is at 0.82 or higher. That's a wide gap, and 0.6 sits comfortably in the middle of it, so I left the threshold where the starter had it rather than moving it for no reason.

Two of my own questions turned out not to be great tests, and I'm noting it here instead of hiding it: "transfer credits toward the major" and "free bike registration" aren't actually covered anywhere in `campus_life` — I wrote them in Milestone 2 without checking the corpus closely enough first. Their best distances (0.571 and 0.621) land right around the cutoff, which is expected once you know they're really out-of-scope material wearing an "in-scope" label. The bike one lands just over 0.6, so the gate refuses it correctly (0 model calls). The transfer-credit one lands just under 0.6, so it gets past the gate — but the model still refused it correctly, because the grounding instruction in `generate.py` caught it as the second layer: none of the five retrieved chunks actually mention transfer credit policy, so it said it didn't have enough information instead of guessing. That's the exact case the two-layer design is for, and I didn't have to change anything to see it work.

## How I Used AI

**1.** For Milestone 3, I'd already decided from reading the corpus in Milestone 1 that `campus_life`'s posts are all short and single-topic, so a post shouldn't get cut apart the way the 800-character fallback would eventually do to a longer one, I wanted paragraph breaks respected instead. I used Claude as a proofreader on the implementation rather than to make that call for me.I described the rule I wanted and had it check my draft logic against edge cases (a document longer than `CHUNK_SIZE`, a document with no blank lines at all). It flagged that my first pass carried more handling than the corpus needed, a whole sentence-splitting fallback for a case that doesn't occur in any of my 88 documents, so I cut that back myself and kept the loop plain.

**2.** For Milestone 4, I ran `python app.py retrieve` myself on all five of my questions and all five `OUT_OF_SCOPE` questions and wrote down the ten distances by hand. I used Claude as a check on my own read of the numbers. I wanted a second opinion on where the gap actually was before I committed to leaving `THRESHOLD` at 0.6 instead of moving it. It agreed the gap was clean (0.24–0.42 vs. 0.82–0.93) but pointed out something I'd missed: two of my own questions, "transfer credits toward the major" and "free bike registration," don't have an answer anywhere in `campus_life` at all. I checked that myself with a grep for "bike" and "transfer" across the documents and confirmed it was right a gap in my own Milestone 2 question-writing, not in retrieval. I decided to keep the cutoff at 0.6 and note the two bad questions rather than quietly swap them out.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
