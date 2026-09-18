# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none, because the grader can't
> read it.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Week 1

## What This Does

The Unofficial Guide is a command-line tool that answers student life questions using the 88-post campus_life corpus covering housing, dining, courses, and campus rules. When a user runs python app.py ask "your question", the system searches for relevant paragraphs within the posts. If no content closely matches the query, it immediately returns "I don't have enough information about that" to prevent hallucinated answers for out-of-scope questions. Otherwise, a language model generates an answer using only the retrieved snippets and explicitly cites the source file for each fact.

## Chunking Strategy

**Chunk size:** I changed how the text is divided. Now, each single paragraph becomes its own 'chunk', and I've pasted the post's title onto the beginning of every chunk. This means I'm not cutting the text at arbitrary character limits. As a result, the 88 original posts were split into 183 chunks. On average, these chunks are 167 characters long, with the shortest being 63 characters and the longest being 397 characters.

**Overlap:** None.

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

The starter cut fixed 800 character windows, and on campus_life that split
nothing at all: the longest post is 549 characters (housing_old_brewhouse.txt),
so 88 documents came out as 88 chunks averaging 317 characters. Reading the
posts showed why that is too coarse. 72 of the 88 have more than one paragraph,
and each paragraph is usually its own topic. Old Brewhouse covers the building's
history, the good, the bad, laundry and noise inside one 549 character post, so
a question about its heating has to match a chunk that is mostly about other
things.

I split on paragraph breaks instead of a character count because these posts are
already written one topic per paragraph, and any character limit would cut
through sentences. I add the title line to every chunk because many paragraphs
never name their subject: "Expect 7 hours a week, plus 3 on lab weeks." does not
say which course, and 32 of my 88 documents name their subject only in the
title. I use no overlap because the title already carries the context, and
overlap would only pull part of the neighbouring topic into each chunk.


## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_cs_340_exams.txt#1` — produced by: `chunker.py::split_documents`

```
CS 340 Databases — assessment

Start the term project in week three, not week eight; everyone learns this the hard way.
```

**Chunk 3** — source: `course_phys_130_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for PHYS 130 Mechanics

People keep asking so: 7 hours a week, plus 3 on lab weeks. That's real time, not optimistic time.
```

**Chunk 4** — source: `dining_verrill_street_grill_followup.txt#1` — produced by: `chunker.py::split_documents`

```
Re: Verrill Street Grill

Also worth saying: one register, so the queue is a single line no matter how busy. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_morrow_house.txt#1` — produced by: `chunker.py::split_documents`

```
Morrow House — what it's actually like

The good: cheapest housing tier by about $900 a year, and the singles are real singles.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** How much does it cost to use a dryer in Morrow House?

**Answer:**

```
  (best distance 0.275, cutoff 0.6)

It costs $1.25 to use a dryer in Morrow House (sources:
housing_morrow_house_laundry.txt and housing_morrow_house.txt).

Sources retrieved: housing_aldridge_hall_laundry.txt,
housing_innisfree_hall_laundry.txt, housing_morrow_house.txt,
housing_morrow_house_laundry.txt, housing_old_brewhouse_laundry.txt
```

The same command on an out-of-scope question is stopped by the gate before the
model runs, so it costs no model call:

```
$ python app.py ask "What is the capital of Mongolia?"
  (best distance 0.787, cutoff 0.6)

I don't have enough information about that.

0 model calls this session
```

**My relevance cutoff:** 0.6

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

I kept the starter's 0.6 after re-measuring, because my Milestone 3 chunker
moved every distance. My five in-corpus questions scored 0.244 to 0.385 and the
five out-of-corpus ones scored 0.787 to 0.923, so the gap runs from 0.385 to
0.787 with nothing inside it and 0.6 sits close to its midpoint of 0.586. I
tried every cutoff from 0.45 to 0.75 and all of them answer 5 of 5 in-corpus
questions and refuse 5 of 5 out-of-corpus ones, so 0.6 needs no change.

| Question | In corpus? | Best distance |
|---|---|---|
| How much printing credit do students get each semester? | yes | 0.385 |
| Which place on campus has real espresso? | yes | 0.366 |
| What is the last week you can drop a course? | yes | 0.332 |
| How much does it cost to use a dryer in Morrow House? | yes | 0.275 |
| How late is the library open during reading week? | yes | 0.244 |
| What is the capital of Mongolia? | no | 0.787 |
| How do I change the oil in a diesel engine? | no | 0.923 |
| Who won the 1994 World Cup? | no | 0.847 |
| What is the recommended dosage of ibuprofen for a headache? | no | 0.849 |
| How do I write a for loop in Rust? | no | 0.860 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** When python app.py index crashed during embedding, Claude traced the error to onnxruntime attempting to use Apple's CoreML provider on an Intel Mac, which cannot run the model. To fix it, Claude pinned the embedder to CPUExecutionProvider in store.py. I then asked for the exact before and after of that single line to report the starter-code issue to the teaching staff rather than just keeping the local patch.

**2.** I had Claude replace split_documents in chunker.py with a paragraph-based chunker that appends each post's title to every chunk. Finally, I verified the output against criterion 4 to confirm that all 183 generated chunks successfully retained their subject names before committing.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Week 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     week 1 — the point is that someone can see what you said before you knew
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
     week — not a new one. Plus a sentence on how you decided. That sentence
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
