# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer. 
<!--MISS-->
<!--My target was "4 of 5". But acreoss all three runs I actually got 3 out of 5, and it was the same two questions failing every single time. Since those 2 questions ask about tings that literally do not exist in the corpus there is nothing to retruve so the system is not behaving inconsistently, the questions are just unanswerable so i say this is a clean miss-->

**Why this target:**
<!-- e.g. "One of my questions is about a topic only two documents mention, so
     I expect that one to be hard." -->

---

## 2. Every answer names a source

Every answer the system produces names at least one source document. 
<!--MISS-->
<!-- This is kind of a side effect of #1, there iis no chunk with the asnwer so the model correctly says i dont have enough information instead of naming a source.o the 2 failures here are the exact same 2 questions as above, for the exact same reason. I'm pointing that out so it's clear this isn't a second, unrelated bug. it's one root cause showing up on two criteria.-->

**Why this target:**
<!-- Why all five and not four? What about your setup makes that achievable —
     or what would have to go wrong for it not to be? -->

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries. 
<!--MET-->
<!--Target was 4/5, I got 5/5, and the numbers aren't even close together. The worst-case out-of-scope distance (0.825) is way above my 0.6 cutoff, and even the worst in-corpus distance (0.621, the bike question) barely goes over it. So this isn't a "just barely passed" situation; there's a wide safety margin.-->


<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**
<!-- What did your distances look like when you set the cutoff in Milestone 4?
     Was there a clean gap, or did the two groups overlap? -->

---

## 4. Something about your chunks

Chunks are complete thoughts.  
<!--MET-->
<!-- I looked at the 5 sample chunks and confirmed none get cut off mid-sentence-->

<!-- At least 4 of 5 sampled chunks read as a complete thought, beginning and ending at a natural boundary (a reply marker, a thread title, or a full sentence) rather than being cut off mid-sentence or mid-word. -->

**Why this target:**
<!--These are informal forum replies so some end on an odd note or skip punctuation even when the chunker did nothing wrong.-->

## 5. something about mixing up similar numbers

Doesn't mix up similar numbers.
<!--MET-->
<!--I read the generated text of all 3 runs and confirmed "eight" was always the degree-total number and "two" was always scoped correctly to "per year," even though the exact sentence wording changed slightly between runs.

The overall pattern: two misses, both traced to the same root cause (bad test questions), and three clean METs with real margin -->


<!--When asked about the pass/fail limit, the answer contains "eight" (the
degree-wide total) and not "two" (the annual limit)-->

**Why this target:**

<!--The pass/fail thread states two numbers close together in the same reply
(2 per year, 8 total across the degree). A system that's sloppy about
grounding could easily grab the wrong one and still sound confident.-->

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
