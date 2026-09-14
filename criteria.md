# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in week 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next week costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**

Two of my questions are hard on purpose: seven laundry documents look almost
identical, and seven housing documents say the library is open until 2am, which
could hide the reading-week hours. I expect one of those to miss, so 5 of 5
would be optimistic.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**

All five, because every excerpt in the prompt is labelled with its filename and
the model is told twice to name the file it used. I judge this on the model's
own answer, not the "Sources retrieved" line the app always prints.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**

My test questions scored 0.320 to 0.427 and the out-of-scope ones 0.825 to
0.934, a clean gap with the 0.6 cutoff in between. I kept 4 of 5 rather than
5 of 5 because any chunking change in Milestone 3 will shift these distances.

---

## 4. Chunks are big enough to keep their subject

No chunk is too small to know what it is about: every chunk keeps the course,
building, dining hall, or topic from its document's title. Among the 32
documents that name their subject only in their first line, zero chunks lose
that subject.

**Why this target:**

32 of my 88 documents name their subject only in the title, so a chunk cut from
the body of "Laundry in Morrow House" would hold a dryer price for no building
at all. I chose zero rather than "most" because each chunk that fails makes
that document's facts impossible to find or attribute.

---

## 5. Answers come from the right document, not a look-alike

When I ask about one specific building or course, the answer uses that
building's or course's facts and cites its document, not a similar-looking one.
At least 4 of these 5 questions get the correct fact and the matching source:

- How much does a dryer cost in Morrow House? ($1.25)
- How do you pay for laundry in Old Brewhouse? (coin only)
- Which floors are quiet floors in Aldridge Hall? (3 and 4)
- How many hours a week does BIOL 160 take? (9 to 11)
- How is MATH 220 curved? (to a B- median)

**Why this target:**

Criterion 2 only checks that a source is named, but 32 of my documents are near
copies (all seven laundry documents share three identical sentences), so
retrieval can easily return the wrong building. I set 4 of 5 because I expect
the look-alikes to cause one miss, and fewer than 4 would mean the system can't
be trusted on the topics my documents say students ask about most.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     WEEK 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in week 2:** For at least 4 of 5 questions, the top three
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
