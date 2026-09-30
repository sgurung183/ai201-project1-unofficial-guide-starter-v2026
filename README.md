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

When I ran `python app.py index` with the starter chunker, it printed 88 documents in and 88 chunks out the 800-character window never actually cut anything, because when I read the corpus in Milestone 1 I saw every post in `campus_life` is short (179–563 characters). Each post is also about exactly one thing: one dorm, one course, one dining hall. The paragraphs inside a post ("the good", "the bad", laundry costs, noise) are different details about that same thing, not different topics.

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
| 1. Retrieved chunk contains the answer | 4 of 5 | 3 of 5 | 3 of 5 | 3 of 5 | MISS |
| 2. Every answer names a source | 5 of 5 | 3 of 5 | 3 of 5 | 3 of 5 | MISS |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Chunks are complete thoughts | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Doesn't mix up similar pass/fail numbers | correct on every try | correct | correct | correct | MET |

Full run data: `results/run_2026-09-23_1630_before.md`, produced by `run_eval.py::main` (retrieval via `store.py::search`, generation via `generate.py`).

**Criteria 1 & 2** — real output, `run_eval.py::main`:

```
How many pages does the printing quota cover?
  best distance 0.4182 (passed the gate)
  "The printing quota covers roughly 600 black-and-white pages per semester (admin_printing_quota.txt)."
  -> chunk contains the answer, names a source

How many pass/fail options can you use throughout your degree?
  best distance 0.2408 (passed the gate)
  "You can use a maximum of eight pass/fail options across a degree (and two per year). Source: admin_pass_fail_option.txt"
  -> chunk contains the answer, names a source

Does the registrar or the department decide if transfer credits count toward the major?
  best distance 0.5711 (passed the gate)
  "I do not have enough information to answer this question."
  -> no chunk contains the answer (not covered anywhere in campus_life), no source named

Is street parking on Verrill legal?
  best distance 0.3605 (passed the gate)
  "Yes, parking on Verrill Street is legal. Source: admin_parking_permits.txt"
  -> chunk contains the answer, names a source

Does campus offer free bike registration?
  best distance 0.6205 (refused by the gate)
  "I don't have enough information about that."
  -> no chunk contains the answer (not covered anywhere in campus_life), no source named
```

3 of 5 questions have a retrieved chunk that contains the answer, and 3 of 5 answers name a source, the same two questions fail both criteria, because they're the two I flagged in Unit 1's Sample Answer section as not actually being covered by `campus_life`. That's a corpus/question-writing problem, not a retrieval or generation bug: retrieval and the gate are both behaving correctly on those two, there's just nothing to find.

**Criterion 3** — real output, `run_eval.py::check_out_of_scope`, cutoff 0.6:

```
Out-of-scope questions (the gate should refuse these):
  refused  (best distance 0.825)  What is the capital of Mongolia?
  refused  (best distance 0.934)  How do I change the oil in a diesel engine?
  refused  (best distance 0.886)  Who won the 1994 World Cup?
  refused  (best distance 0.844)  What is the recommended dosage of ibuprofen for a headache?
  refused  (best distance 0.896)  How do I write a for loop in Rust?
  -> gate refused 5 of 5
```

**Criterion 4** — real output, `chunker.py::split_documents` (see Unit 1's Sample Chunks for the full text): all 5 sampled chunks begin and end at a post boundary — a title line through a full final sentence — none are cut mid-sentence or mid-word.

**Criterion 5** — real output, `generate.py`, all 3 runs of the pass/fail question:

```
run 1: "You can use a maximum of eight pass/fail options across a degree (and two per year)."
run 2: "You can use a maximum of eight pass/fail options across a degree (with a maximum of two per year)."
run 3: "You can use a maximum of eight pass/fail options across a degree (with a maximum of two per year)."
```

Every run leads with "eight" as the degree-wide total and keeps "two" scoped to "per year" — it never swaps them.

## Verdicts

CRITERION 1: For at least 4 of my 5 test questions, the retrieved chunks include one thatcontains the answer. 
VERDICT : MISS
REASON: My target was "4 of 5". But acreoss all three runs I actually got 3 out of 5, and it was the same two questions failing every single time. Since those 2 questions ask about tings that literally do not exist in the corpus there is nothing to retruve so the system is not behaving inconsistently, the questions are just unanswerable so i say this is a clean miss

CRITERION 2: Every answer the system produces names at least one source document. 
VERDICT: MISS
REASON: This is kind of a side effect of #1, there iis no chunk with the asnwer so the model correctly says i dont have enough information instead of naming a source.o the 2 failures here are the exact same 2 questions as above, for the exact same reason. I'm pointing that out so it's clear this isn't a second, unrelated bug. it's one root cause showing up on two criteria.

CRITERION 3: When I ask a question my documents clearly don't cover, the relevance gate stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries. 
VERDICT: MET
REASON: Target was 4/5, I got 5/5, and the numbers aren't even close together. The worst-case out-of-scope distance (0.825) is way above my 0.6 cutoff, and even the worst in-corpus distance (0.621, the bike question) barely goes over it. So this isn't a "just barely passed" situation; there's a wide safety margin.

CRITERION 4: Chunks are complete thoughts.  
VERDICT: MET-->
REASON:I looked at the 5 sample chunks and confirmed none get cut off mid-sentence. At least 4 of 5 sampled chunks read as a complete thought, beginning and ending at a natural boundary (a reply marker, a thread title, or a full sentence) rather than being cut off mid-sentence or mid-word. 

CRITERION 5: Doesn't mix up similar numbers.
VERDICT: MET
REASON: I read the generated text of all 3 runs and confirmed "eight" was always the degree-total number and "two" was always scoped correctly to "per year," even though the exact sentence wording changed slightly between runs.


## Diagnoses

Here's the Diagnoses text — tell me if you want any part adjusted, then you can paste it in yourself:

---

## Criteria 1 & 2 (retrieved chunk contains the answer / every answer names a source)

Both misses have the same root cause and it isn't a bug in any of the five pipeline stages. It's a **loading**-stage gap: two of my five test questions ("transfer credits toward the major," "free bike registration") ask about topics that never made it into the corpus in the first place. I confirmed this with a plain grep for "bike" and "transfer" across all 88 documents in `campus_life` and got zero hits for either topic.

Given that, retrieval is doing exactly what it should: for the transfer-credits question it returns the five closest chunks it has (0.571 distance, closer than any true out-of-scope question, but still not a real match), and for the bike question it returns chunks close enough to pass the gate at first glance but ultimately unrelated (0.621). Chunking and embedding aren't implicated either, the chunks retrieved are well-formed and correctly embedded, they just don't answer the question because no chunk anywhere does. And generation behaves correctly too: for both questions it refuses rather than fabricating an answer, because the grounding instruction in `generate.py` checks whether the retrieved chunks actually contain the claim before answering.

So this is one problem, not two: a question-writing gap from Milestone 2, where I wrote test questions without first checking they were answerable from the corpus. Nothing downstream of that (chunking, embedding, retrieval, generation) needs fixing. they're all behaving as designed on data that was never there to find.

## Criteria 3, 4, 5:
 no misses, nothing to diagnose. If I were tightening a target, criterion 3 (gate stops out-of-corpus questions) has the most slack, 5/5 against a target of 4/5, with a wide margin between in and out-of-scope distances so that's the one I'd raise, to "5 of 5" instead of "4 of 5," if I were rewriting targets.

## The Improvement

**What I changed:** 
Lowered the relevance gate cutoff in config.py from 0.6 to 0.55.

**Why I picked it:**
My diagnosis found that criteria 1 and 2 both fail because two of my test questions ask about topics that aren't anywhere in the corpus, not because of a chunking, retrieval, or gate problem. The transfer-credits question sat right at 0.571, close enough to the old 0.6 cutoff that it seemed worth testing whether moving the threshold changed anything. It's a small, single-number change that directly tests the borderline case my diagnosis flagged, without touching multiple parts of the pipeline at once.

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 3 of 5 | 3 of 5  | 3 of 5  | MISS |
| 2. Every answer names a source | 5 of 5 | 3 of 5 | 3 of 5 | 3 of 5 | MISS  |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5  | 5 of 5  | MET |
| 4. Chunks are complete thoughts| 4 of 5| 5 of 5| 5 of 5| 5 of 5 |MET |
| 5.Doesn't mix up similar pass/fail numbers |correct on all try | | | | MET |

**Did it help?**

No, it didn't really change anything. I lowered the cutoff from 0.6 to 0.55 hoping it might catch the transfer-credits question in a different way, since it sat right at 0.571. It did shift where that question gets refused (the gate itself now, instead of the model), but the final answer is the same either way. All five criteria came out exactly the same as before: 3/5, 3/5, 5/5, 5/5, correct every time. Makes sense in hindsight, the answers just aren't in my corpus, so no cutoff was ever going to fix that.

## What's Still Broken

Criteria 1 and 2 are still broken, 3/5 instead of the 4/5 and 5/5 I wanted. Both come down to the same two questions (transfer credits, bike registration) asking about stuff that just isn't in the corpus. That's not something a threshold tweak, chunking change, or prompt fix can touch, the content has to exist first. To actually fix it I'd need to add real source documents on those two topics and reindex, which I didn't do here since it felt outside what this milestone was asking for. If I kept going I'd add those two docs, or swap the two bad questions for ones the corpus actually covers, and rerun to see if I hit 4/5 and 5/5.

## What I'd Do Differently

I'd write my Milestone 2 questions by grepping the corpus first, not after. Two of my five questions turned out to ask about things that don't exist anywhere in campus_life, and I only caught that in Unit 1 while double checking the relevance gate, not when I wrote them. If I'd checked upfront, my whole Unit 2 run would've had five real, answerable questions instead of two dead ones dragging down criteria 1 and 2 the whole time. I'd also probably write criterion 1 and 2 as one combined criterion next time, since in this system they always fail together for the same reason and tracking them separately didn't tell me anything extra.
