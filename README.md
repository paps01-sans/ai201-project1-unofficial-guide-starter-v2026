# The Unofficial Guide

**Author:** Kidus A
**Corpus:** `campus_life`

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

The Unofficial Guide makes short campus-life posts searchable through plain
questions. This project uses the `campus_life` corpus, which covers housing,
dining, courses, registration, and other student experiences. It retrieves the
most relevant document chunks, refuses questions that fall outside the corpus,
and asks the model to answer from the retrieved text while naming its source.

## Chunking Strategy

**Chunk size:** 400 characters maximum
**Overlap:** 0 characters

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

I chose paragraph-based chunks because the campus_life documents are short
posts of one to three paragraphs, and each paragraph usually contains one
complete answer or thought. I attach each title to the first content paragraph
and keep later paragraphs separate, so related details do not get buried in a
whole-document chunk. The 400-character maximum is above the longest observed
paragraph, and zero overlap avoids repeating complete paragraph boundaries.

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
Start the term project in week three, not week eight; everyone learns this the hard way.
```

**Chunk 3** — source: `course_phys_130_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for PHYS 130 Mechanics

People keep asking so: 7 hours a week, plus 3 on lab weeks. That's real time, not optimistic time.
```

**Chunk 4** — source: `dining_verrill_street_grill_followup.txt#1` — produced by: `chunker.py::split_documents`

```
Also worth saying: one register, so the queue is a single line no matter how busy. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_morrow_house.txt#1` — produced by: `chunker.py::split_documents`

```
The good: cheapest housing tier by about $900 a year, and the singles are real singles.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** Is the housing lottery random for juniors and seniors?

**Answer:** No, for juniors and seniors, the housing lottery orders students by
accumulated credit hours first, and only uses a random tie-break
(`admin_housing_lottery.txt`).

```

```

**My relevance cutoff:**

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question                                                                | In corpus? | Best distance |
| ----------------------------------------------------------------------- | ---------- | ------------- |
| Is the housing lottery random for juniors and seniors?                  | Yes        | 0.1355        |
| How long are the peak lunch wait times at Pellew Dining Hall?           | Yes        | 0.1725        |
| How many hours per week should students expect to spend outside CS 210? | Yes        | 0.2846        |
| How much does laundry cost in Aldridge Hall?                            | Yes        | 0.2585        |
| Which floors in Aldridge Hall are quiet floors?                         | Yes        | 0.2500        |
| What is the capital of Mongolia?                                        | No         | 0.7873        |
| How do I change the oil in a diesel engine?                             | No         | 0.9228        |
| Who won the 1994 World Cup?                                             | No         | 0.8474        |
| What is the recommended dosage of ibuprofen for a headache?             | No         | 0.8243        |
| How do I write a for loop in Rust?                                      | No         | 0.8768        |

**My cutoff:** 0.6. The in-corpus questions ranged from 0.1355 to 0.2846,
while the out-of-corpus questions ranged from 0.7873 to 0.9228. I kept the
starter cutoff because it sits comfortably in the gap and rejects the clearly
unrelated questions. I kept `TOP_K = 5` because the relevant chunk appeared
in the first five results for each inspected question.

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** I asked Copilot to inspect the `campus_life` documents and recommend a
chunking strategy that matched their structure. It identified that every
document has multiple short paragraphs and suggested keeping paragraph
boundaries instead of using the starter's fixed 800-character windows. I
implemented paragraph-based chunks with a 400-character maximum, zero overlap,
and the document title attached to the first content paragraph.

**2.** I asked Copilot to compare the before-run results and suggest one fix
that matched the evidence. It noticed that the test questions use exact names,
numbers, and course terms, so I added BM25 keyword ranking beside semantic
search in `store.py::search`. After the change, the exact laundry source moved
to the top of the results, but the overall scores stayed the same because the
before run was already 5/5.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

**Stretch feature:** Metadata filtering by source filename. I am adding a
`--source FILENAME` option to `retrieve` and `ask` so a user can narrow
retrieval to one document before the relevance gate and answer generation.
The corpus has no reliable dates, so this feature filters by source only.

For example, `python app.py retrieve "Is the housing lottery random for
juniors and seniors?" --source admin_housing_lottery.txt` returns only the
matching housing-lottery chunk. An unknown source returns no chunks instead of
silently searching the whole corpus.

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

| Criterion                                       | Target | Run 1 | Run 2 | Run 3 | Verdict |
| ----------------------------------------------- | ------ | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer          | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 2. Every answer names a source                  | 5 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 3. Gate stops out-of-corpus questions           | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 4. Sampled chunks are complete thoughts         | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 5. Expected phrase and supporting source appear | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

### Criterion 1: Retrieved chunk contains the answer

Evidence from `results/run_2026-09-30_1504_before.md`, produced by
`run_eval.py::run_once` using `store.py::search`:

```
No, juniors and seniors are ordered by accumulated credit hours first, and they are only tie-broken randomly (admin_housing_lottery.txt).
The peak lunch wait times at Pellew Dining Hall are 12 to 18 minutes (dining_pellew_dining_hall.txt and dining_pellew_dining_hall_followup.txt).
Students should expect to spend 8 to 10 hours a week outside class for CS 210 (course_cs_210.txt and course_cs_210_workload.txt).
Laundry in Aldridge Hall costs $1.75 for a wash and $1.50 for a dry (housing_aldridge_hall_laundry.txt and housing_aldridge_hall.txt).
Floors 3 and 4 in Aldridge Hall are quiet floors. (housing_aldridge_hall_noise.txt)
```

### Criterion 2: Every answer names a source

Evidence from `results/run_2026-09-30_1504_before.md`, produced by
`run_eval.py::run_once` and `generate.py::answer_from_chunks`:

```
(admin_housing_lottery.txt)
(dining_pellew_dining_hall.txt and dining_pellew_dining_hall_followup.txt)
(course_cs_210.txt and course_cs_210_workload.txt)
(housing_aldridge_hall_laundry.txt and housing_aldridge_hall.txt)
(housing_aldridge_hall_noise.txt)
```

### Criterion 3: Gate stops out-of-corpus questions

Evidence from `results/run_2026-09-30_1504_before.md`, produced by
`run_eval.py::check_out_of_scope` using `gate.py::check`:

```
What is the capital of Mongolia? | 0.787 | refused
How do I change the oil in a diesel engine? | 0.923 | refused
Who won the 1994 World Cup? | 0.847 | refused
What is the recommended dosage of ibuprofen for a headache? | 0.824 | refused
How do I write a for loop in Rust? | 0.877 | refused
-> gate refused 5 of 5
```

### Criterion 4: Sampled chunks are complete thoughts

Evidence from the five chunks printed in this README's Unit 1 sample, produced
by `chunker.py::split_documents`:

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

Start the term project in week three, not week eight; everyone learns this the hard way.

Workload for PHYS 130 Mechanics

People keep asking so: 7 hours a week, plus 3 on lab weeks. That's real time, not optimistic time.

Also worth saying: one register, so the queue is a single line no matter how busy. Nobody tells you this at orientation.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.
```

### Criterion 5: Expected phrase and supporting source appear

Evidence from `results/run_2026-09-30_1504_before.md`, produced by
`run_eval.py::run_once` and `generate.py::answer_from_chunks`:

```
No, juniors and seniors are ordered by accumulated credit hours first, and they are only tie-broken randomly (admin_housing_lottery.txt).
The peak lunch wait times at Pellew Dining Hall are 12 to 18 minutes (dining_pellew_dining_hall.txt and dining_pellew_dining_hall_followup.txt).
Students should expect to spend 8 to 10 hours a week outside class for CS 210 (course_cs_210.txt and course_cs_210_workload.txt).
Laundry in Aldridge Hall costs $1.75 for a wash and $1.50 for a dry (housing_aldridge_hall_laundry.txt and housing_aldridge_hall.txt).
Floors 3 and 4 in Aldridge Hall are quiet floors. (housing_aldridge_hall_noise.txt)
```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| #   | Criterion                                    | Verdict | How I decided                                                                                                                                            |
| --- | -------------------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Retrieved chunk contains the answer          | MET     | Each of the three runs had the expected answer in the retrieved evidence for all five questions, so 5/5 met the target of at least 4/5 every time.       |
| 2   | Every answer names a source                  | MET     | All 15 generated answers included at least one source filename, so every run reached 5/5 against the target of 5/5.                                      |
| 3   | Gate stops out-of-corpus questions           | MET     | The deterministic gate refused all five out-of-corpus questions, so its 5/5 result exceeded the target of at least 4/5 in each run column.               |
| 4   | Sampled chunks are complete thoughts         | MET     | All five sampled chunks read as complete thoughts without a sentence cut off at either end, meeting the target of at least 4/5.                          |
| 5   | Expected phrase and supporting source appear | MET     | Every generated answer contained its expected phrase and named the supporting document, giving 5/5 in all three runs against the target of at least 4/5. |

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

I missed nothing in the before run. There is therefore no failed question or
pipeline stage to diagnose: the loading stage supplied the campus_life
documents, the paragraph chunker produced complete sampled thoughts, and
embedding and retrieval returned answer-containing chunks for all five
questions. Generation then included the expected phrase and a source filename
in every one of the 15 answers. The gate also refused all five out-of-scope
questions, so there is no failure pattern across the five stages.

The targets were somewhat safe. I would tighten criterion 1 from "at least 4
of 5" to "5 of 5" because every test question has a concrete answer in this
corpus and the current retrieval evidence reached 5/5 in all three runs. I
would keep criterion 2 at 5 of 5 because it already requires perfection. The
other 4-of-5 targets are reasonable tolerance targets for generation and gate
behavior, but this result does not prove they would hold on a new question set.

## The Improvement

**What I changed:** I added BM25 keyword ranking to `store.py::search` and
combined it with the existing semantic ranking using reciprocal rank fusion.
The original semantic distance is still used by the relevance gate.

**Why I picked it:** The before run missed nothing, but criterion 1 was the
safest target and the questions include exact names, numbers, and terms such
as "CS 210" and "$1.75". Hybrid retrieval tests whether keyword matching
makes those exact facts easier to retrieve without changing chunking,
generation, or the cutoff.

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion                                       | Target | Run 1 | Run 2 | Run 3 | Verdict |
| ----------------------------------------------- | ------ | ----- | ----- | ----- | ------- |
| 1. Retrieved chunk contains the answer          | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 2. Every answer names a source                  | 5 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 3. Gate stops out-of-corpus questions           | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 4. Sampled chunks are complete thoughts         | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |
| 5. Expected phrase and supporting source appear | 4 of 5 | 5/5   | 5/5   | 5/5   | MET     |

After-run evidence from `results/run_2026-09-30_1624_after.md`, produced by
`run_eval.py::run_once`, `run_eval.py::check_out_of_scope`, and
`generate.py::answer_from_chunks`:

```
Is the housing lottery random for juniors and seniors? | pass | pass | pass
How long are the peak lunch wait times at Pellew Dining Hall? | pass | pass | pass
How many hours per week should students expect to spend outside CS 210? | pass | pass | pass
How much does laundry cost in Aldridge Hall? | pass | pass | pass
Which floors in Aldridge Hall are quiet floors? | pass | pass | pass

What is the capital of Mongolia? | 0.787 | refused
How do I change the oil in a diesel engine? | 0.923 | refused
Who won the 1994 World Cup? | 0.847 | refused
What is the recommended dosage of ibuprofen for a headache? | 0.824 | refused
How do I write a for loop in Rust? | 0.877 | refused
-> gate refused 5 of 5

No, juniors and seniors are ordered by accumulated credit hours first, and only tie-break randomly (admin_housing_lottery.txt).
The peak lunch wait times at Pellew Dining Hall are 12 to 18 minutes. (dining_pellew_dining_hall.txt)
Students should expect to spend 8 to 10 hours a week outside class for CS 210 (course_cs_210.txt, course_cs_210_workload.txt).
Laundry in Aldridge Hall costs $1.75 to wash and $1.50 to dry (housing_aldridge_hall_laundry.txt and housing_aldridge_hall.txt).
The quiet floors in Aldridge Hall are floors 3 and 4 (housing_aldridge_hall_noise.txt).
```

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

It preserved the existing results but did not improve the aggregate scores:
the before and after evaluations were both 5/5 for every criterion in all
three runs, and both gate checks refused 5/5 out-of-scope questions. It did
improve the retrieval ordering for the laundry question by putting the exact
`housing_aldridge_hall_laundry.txt` source first, but the baseline already
retrieved enough evidence, so the measured verdicts stayed the same.

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

None of the five criteria are still missed. The main limitation is that the
test set is small, so passing 5/5 does not prove the system will work for every
campus question. I would test more questions, especially ones with uncommon
names or numbers, to see if the hybrid search helps outside this set. I also
would compare the keyword and semantic rankings on those new questions before
changing the cutoff or adding another fix. I stopped here because the required
before and after tests both met every target, and I was only supposed to make
one improvement.

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->

Next time I would make criterion 1 stricter and require 5 of 5 instead of 4 of 5. All five questions had clear answers in the corpus, so 4 of 5 was probably
too easy for this test set. I would also write criterion 4 with a bigger,
specified sample instead of only five chunks, since five chunks can make the
chunking result look better than it really is. The other criteria still made
sense, but I would test them with more questions before trusting the results.
