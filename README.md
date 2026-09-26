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

Results file: `results/run_2026-09-26_1146_before.md`, produced by
`run_eval.py::main` (3 runs per question, caching off).

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks are big enough to keep their subject | 0 lose it | 0/183 | 0/183 | 0/183 | MET |
| 5. Answers come from the right document, not a look-alike | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |

Criteria 1, 3 and 4 are measured by retrieval and chunking, which are
deterministic, so one pass is the whole measurement and the same number goes in
all three columns. Criteria 2 and 5 depend on the generated answer and were
measured on all three runs.

How I judged criterion 5, since there is no scorer.py yet: an answer passes if
it states the correct fact and cites the document for the building or course
asked about. I applied that by reading, so run 2's "with coins only" counts the
same as "coin only". A literal string match would have scored that run 3 of 5,
which is a fact about my string matching and not about the system.

### Criterion 1: retrieved chunk contains the answer

Source: Run 1, `store.py::search`. The check is whether the `expects` phrase
from `questions.py` appears in any of the five retrieved chunks.

```
  PASS  $30        How much printing credit do students get each semester?
        found in: admin_printing_quota.txt#0
  PASS  Ridgeway   Which place on campus has real espresso?
        found in: dining_the_ridgeway_cafe.txt#0
  PASS  six        What is the last week you can drop a course?
        found in: admin_add_drop_deadline.txt#0, admin_withdrawal_deadline.txt#0
  PASS  $1.25      How much does it cost to use a dryer in Morrow House?
        found in: housing_morrow_house_laundry.txt#0, housing_morrow_house.txt#3
  PASS  10pm       How late is the library open during reading week?
        found in: study_library_hours.txt#0
  -> 5/5
```

### Criterion 2: every answer names a source

Source: Run 2, `generate.py::answer_from_chunks`. All 15 answers named at least
one file in the model's own text. Two of them:

```
Students get $30 of printing per semester.

Source: admin_printing_quota.txt
```

```
It costs $1.25 to use a dryer in Morrow House (housing_morrow_house.txt and
housing_morrow_house_laundry.txt).
```

### Criterion 3: the gate stops out-of-corpus questions

Source: Run 1, `run_eval.py::check_out_of_scope` calling `gate.py::check`,
cutoff 0.6. No model call was made for any of these.

```
| What is the capital of Mongolia? | 0.787 | refused |
| How do I change the oil in a diesel engine? | 0.923 | refused |
| Who won the 1994 World Cup? | 0.847 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.849 | refused |
| How do I write a for loop in Rust? | 0.860 | refused |
-> gate refused 5 of 5
```

### Criterion 4: chunks are big enough to keep their subject

Source: `chunker.py::split_documents`. Every chunk from the 32 documents that
name their subject only in the title was checked for that subject.

```
  183 chunks total; 64 from the 32 title-only documents
  chunks that lost their subject: 0
```

### Criterion 5: answers come from the right document, not a look-alike

Source: Run 1, `store.py::search` then `generate.py::answer_from_chunks`. Four
of five passed on every run, and the same question failed all three times.

```
[PASS] How much does a dryer cost in Morrow House?  (dist 0.275)
  A dryer in Morrow House costs $1.25.

  Source: housing_morrow_house_laundry.txt (also found in housing_morrow_house.txt)

[PASS] How do you pay for laundry in Old Brewhouse?  (dist 0.233)
  Laundry in Old Brewhouse is coin only.

  Source: housing_old_brewhouse.txt (and housing_old_brewhouse_laundry.txt)

[PASS] Which floors are quiet floors in Aldridge Hall?  (dist 0.251)
  Floors 3 and 4 are the quiet floors in Aldridge Hall (housing_aldridge_hall_noise.txt).

[PASS] How many hours a week does BIOL 160 take?  (dist 0.301)
  BIOL 160 takes 9 to 11 hours a week (source: course_biol_160.txt and course_biol_160_workload.txt).

[FAIL] How is MATH 220 curved?  (dist 0.614)
  I don't have enough information about that.
```

The failure is not a wrong building or a wrong course. The best chunk for "How
is MATH 220 curved?" sits at 0.614, just above my 0.6 cutoff, so the gate
refused a question my corpus does answer: course_math_220_exams.txt says the
course is "Curved to a b- median".

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
| 1 | Retrieved chunk contains the answer | MET | Target was 4 of 5 and all three runs hit 5 of 5. Retrieval is deterministic, so the same questions return the same chunks every run. |
| 2 | Every answer names a source | MET | Target was every answer, and all 15 answers across the three runs named at least one file in the model's own text, not just in the "Sources retrieved" line the app prints. |
| 3 | Gate stops out-of-corpus questions | MET | Target was 4 of 5 and the gate refused 5 of 5, with the closest out-of-corpus distance at 0.787 against a 0.6 cutoff. |
| 4 | Chunks are big enough to keep their subject | MET | Target was zero failures, and zero of the 64 chunks from my 32 title-only documents lost their subject. |
| 5 | Answers come from the right document, not a look-alike | MET | Target was 4 of 5 and every run came out 4 of 5, so it holds on all three runs rather than just some. It only just holds: the same question fails every time, and it fails because the gate refuses it at 0.614, not because a look-alike document won. |

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

No criterion came out MISSED. All five held on all three runs, so what follows
is a diagnosis of the one question that failed inside a criterion that still
met its target, plus an honest look at whether my targets were set too low.

### The one real failure

**Criterion:** 5. Answers come from the right document, not a look-alike.

**Failing question:** "How is MATH 220 curved?" It failed on all three runs.

**What came back:** "I don't have enough information about that." The corpus
does answer this. course_math_220_exams.txt says the course is "Curved to a b-
median."

**Stage:** embedding, showing up at the gate in retrieval.

**Mechanism:** the question names the course by its bare code, and the
all-MiniLM-L6-v2 embedding carries a code like "MATH 220" weakly compared with
words that have ordinary meaning. Retrieval still ranked correctly: the top two
chunks are both MATH 220, so no look-alike document won. What failed is the
absolute distance. The best chunk sits at 0.6142, my cutoff is 0.6, and the gate
compares the absolute number rather than the ranking, so it refused a question
whose answer was sitting in the number one result.

**Evidence:** the top of the ranking for that question, from store.py::search.

```
1. 0.6142  course_math_220.txt#0        MATH 220 Linear Algebra / I lived here my sophomore year...
2. 0.6156  course_math_220_exams.txt#0  MATH 220 Linear Algebra — assessment / Two midterms and a
                                        cumulative final. Curved to a b- median.
3. 0.6342  course_cs_210.txt#0          CS 210 Data Structures / ...
```

Adding the course name to the same question drops the distance below the cutoff
and the question is answered:

```
0.6142  REFUSED  'How is MATH 220 curved?'
0.4544  passes   'How is MATH 220 Linear Algebra curved?'
0.3615  passes   'How is MATH 220 Linear Algebra graded?'
```

My Milestone 3 chunker did not cause this. Rebuilding the index with the
starter's fallback_split and asking the same question gives 0.6196, slightly
worse than my 0.6142, so both chunkers land above the cutoff.

### The pattern

This is one root cause, not one unlucky question. Asking "How is X curved?" for
all nine courses in the corpus shows bare course codes sitting far higher than
the same questions with the course name attached:

```
bare code:       0.3868 to 0.6142   (1 of 9 refused: MATH 220 at 0.6142)
code plus name:  0.2305 to 0.4544   (0 of 9 refused)
```

MATH 220 is simply the one that crosses the line. The same mechanism produces a
second failure my five criterion 5 questions never caught: "How is CS 210
curved?" returns course_cs_340_exams.txt as its top result at 0.5384, which is a
wrong course answered confidently because it passes the gate. So the pattern is
short questions that identify a course or building by code alone, where the
embedding has little to work with and distances drift toward the cutoff.

### Were my targets set too low?

Partly, yes. Criterion 4 is the weakest of the five. It passes by construction,
because my chunker puts the title on every chunk, so zero failures was
guaranteed the moment the code was written. It is a useful regression guard
against a future chunking change, but it did not test anything this week.
Criteria 1 and 3 also passed with room to spare: criterion 1 came out 5 of 5
against a target of 4 of 5, with the answer ranked first every time, and
criterion 3 refused 5 of 5 with the closest out-of-corpus question at 0.787
against a 0.6 cutoff.

The one I would tighten is criterion 5, from 4 of 5 to 5 of 5. It is the only
criterion that exposed a real defect, and holding it at 4 of 5 lets a permanent
failure sit inside a passing score. I would also make criterion 3 harder by
replacing questions from a different world entirely with near misses, such as
asking about a building that is not in my corpus, since those would land near
the cutoff instead of 0.2 above it.

## The Improvement

**What I changed:** I raised the relevance cutoff in config.py from 0.6 to
0.65. That is the only change. The chunker, the prompt, top-k and the questions
are all exactly as they were for the before run.

**Why I picked it:** My diagnosis above found one question that failed on all
three runs, "How is MATH 220 curved?", and the reason was not a wrong document
winning. The right chunk was ranked first, but its distance was 0.6142 and my
cutoff was 0.6, so the gate threw away a question the corpus answers. My own
measurements showed an empty gap between 0.385 and 0.787, so moving the cutoff
to 0.65 lets that question through while still leaving a wide margin before the
closest out-of-corpus question at 0.787.

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

Results file: `results/run_2026-09-27_0058_after.md`, produced by
`run_eval.py::main` with the cutoff at 0.65.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks are big enough to keep their subject | 0 lose it | 0/183 | 0/183 | 0/183 | MET |
| 5. Answers come from the right document, not a look-alike | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

**Did it help?** Yes, and only where I expected it to. Criterion 5 went from
4 of 5 on every run to 5 of 5 on every run, because the MATH 220 question is now
answered instead of refused, and the answer is correct: "MATH 220 is curved to a
B- median (source: course_math_220.txt and course_math_220_exams.txt)." Nothing
else moved. Criteria 1 and 4 are decided by retrieval and chunking, which this
change does not touch, and criterion 2 stayed at 15 answers out of 15 naming a
file. Criterion 3 also held, because the closest out-of-corpus question sits at
0.787 and the cutoff is still well below it, so the gate refused all five.

The honest limit of this fix is that it treats the symptom. The real problem is
that questions naming a course only by its code embed poorly, and moving the
cutoff does nothing about that. It also spends some of my safety margin: the
distance between my cutoff and the nearest out-of-corpus question shrank from
0.187 to 0.137. I accepted that because the gap I measured is wide and empty,
but a corpus with a narrower gap would not have room for this fix.

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

After the fix, no criterion is still MISSED. All five met their targets on all
three runs of the after log, so the per-criterion template below has no entries
for a missed criterion. Two real defects survive anyway, and both sit inside
criterion 5, which passes only because my five questions do not happen to cover
them. I am writing them up in the same format rather than claiming nothing is
left.

**Defect 1: questions that name a course by its code alone**

- **Criterion:** 5. Answers come from the right document, not a look-alike.
- **Current result:** 5 of 5 on all three after runs, against a target of 4 of
  5. It passes, but only because raising the cutoff pulled the one failing
  question back over the line.
- **Likely cause:** embedding. The all-MiniLM-L6-v2 model carries a code like
  "MATH 220" weakly next to ordinary words, so the distance stays high even when
  the ranking is right. Across all nine courses, "How is X curved?" gives
  distances of 0.387 to 0.614, while the same question with the course name
  attached gives 0.231 to 0.454.
- **What I would do next:** put both the course code and the course name into
  every chunk of a course document, so "MATH 220" and "Linear Algebra" are both
  present whichever one the question uses. The same idea applies to buildings.
- **Why I stopped:** that is a chunker change, and it would move every distance
  in the corpus, so the cutoff would need re-measuring and both run logs would
  need redoing. I had already made one change this week and wanted the before
  and after runs to stay comparable. Changing two things at once would have left
  me unable to say which one helped.

**Defect 2: a confident wrong answer about a course I did not test**

- **Criterion:** 5. Answers come from the right document, not a look-alike.
- **Current result:** not measured by my five questions, so it does not appear
  in any score. "How is CS 210 curved?" returns course_cs_340_exams.txt as its
  closest chunk at 0.5384, which passed the gate before my change and still
  passes now.
- **Likely cause:** the same embedding weakness as defect 1, but worse in
  effect. Here the bare code does not push the distance over the cutoff, it just
  lets a different course rank first, so the system answers confidently from the
  wrong document instead of refusing.
- **What I would do next:** the same chunker change, and then widen my criterion
  5 question set so it covers a course whose look-alike is a different course
  rather than a different building.
- **Why I stopped:** I found this while diagnosing, after my questions and
  targets were already committed, and the week 2 rules say the original
  criterion stays as written. Swapping in new questions mid-week would have
  changed what the before and after numbers even mean.

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->

Criterion 4 is the one I would rewrite. It asks that no chunk lose the subject
named in its document title, and my chunker puts that title on every chunk, so
the criterion could not fail once the code was written. It is a useful guard
against a future chunking change, but it told me nothing this week. I would
replace it with something my current pipeline could actually fail, such as
requiring that a chunk contain the whole of any fact it mentions, since a fact
split across two chunks is the failure mode that chunking really risks.

Criterion 5 is the one I would tighten, from 4 of 5 to 5 of 5. It was the only
criterion that found a real defect, and at 4 of 5 a permanent failure was able
to sit inside a passing score for three straight runs. After the fix it reaches
5 of 5 anyway, so the stricter target costs nothing now and would catch the next
regression.

I would also change criterion 2 and criterion 3. Criterion 2 only asks that an
answer name a source, which every answer did on all six runs, so it never
discriminated. It should ask that the answer name the file that actually
contains the fact. Criterion 3 uses out-of-corpus questions from a different
world entirely, and they scored 0.787 and above against a cutoff of 0.65, so the
gate was never under pressure. Questions about a building or a course my corpus
does not include would land much closer to the line and would be a real test.
