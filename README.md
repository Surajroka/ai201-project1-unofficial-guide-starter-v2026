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

I used a maximum chunk size of approximately 600 characters with no fixed character overlap. Instead of splitting documents at arbitrary character positions, my `split_documents` function groups complete paragraphs and replies together until adding another paragraph would exceed the target size.

I chose this strategy because the `advice_threads` corpus consists of questions followed by individual student replies. Keeping paragraph and reply boundaries intact preserves complete thoughts and prevents sentences from being cut in half. The starter's fixed-size chunker produced 26 chunks with a shortest chunk of only 2 characters. My strategy produced 27 chunks averaging 462 characters, with the shortest chunk at 124 characters and the longest at 598.

I manually inspected five chunks produced by `chunker.py::split_documents`. All five contained enough context to understand at least one piece of student advice without needing the previous or next chunk.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `` — produced by: ``

```
```

Chunk 1

Source: thread_bike_commute.txt
Produced by: chunker.py::split_documents

THREAD: Is a bike worth it for a 20 minute walk commute?

--- reply 1 (14 votes) ---
Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.

--- reply 2 (9 votes) ---
Counterpoint, I sold mine. Between November and March the paths are either icy or salted and salt destroys a drivetrain in one season.

--- reply 3 (22 votes) ---
Both true. I keep a cheap bike for September to November and walk the rest of the year. Total cost was about $120 for the bike and I don't care what happens to it.

Chunk 2

Source: thread_first_gen.txt
Produced by: chunker.py::split_documents

THREAD: Anything specific for first-generation students?

--- reply 1 (33 votes) ---
The advising office has a specific programme and it is genuinely good, but it is opt-in and badly publicised. Ask for it by name.

--- reply 2 (41 votes) ---
The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.

--- reply 3 (16 votes) ---
Emergency fund for textbooks and travel exists and is not means-tested beyond a short form.

Chunk 3

Source: thread_laptop_specs.txt
Produced by: chunker.py::split_documents

THREAD: How much laptop do I actually need for CS courses?

--- reply 1 (31 votes) ---
Less than the recommended spec page says. 16GB of RAM is the one number worth paying for; everything else you'll never notice.

--- reply 2 (18 votes) ---
Adding: the lab machines exist and are better than anything you'll buy. For the heavy assignments people just use those.

--- reply 3 (12 votes) ---
I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.

Chunk 4

Source: thread_office_hours_etiquette.txt
Produced by: chunker.py::split_documents

THREAD: Is it weird to go to office hours with no specific question?

--- reply 1 (44 votes) ---
No, and this is the single most common thing first years get wrong. 'I'm following the lectures but I don't feel like I understand the shape of it' is a completely normal thing to say.

--- reply 2 (29 votes) ---
They're usually empty. You are doing the instructor a favour by turning up.

--- reply 3 (18 votes) ---
If it helps, treat it as a standing appointment. Go every week for a month and it stops feeling like a thing.

Chunk 5

Source: thread_professor_email.txt
Produced by: chunker.py::split_documents

THREAD: Do professors actually answer email?

--- reply 1 (21 votes) ---
Varies enormously. General rule I've found: if the syllabus states a response window, it's honoured. If it doesn't, assume 48 hours and don't panic before then.

--- reply 2 (33 votes) ---
Office hours are dramatically more effective than email for anything that takes more than two sentences to answer. They're also usually empty.

--- reply 3 (15 votes) ---
Empty office hours is the biggest unused resource here and I say that having wasted a year not going

## Sample Answer

**Question:** How much RAM do students recommend for a laptop used for CS courses?

**Answer:** Students recommend 16GB of RAM for a laptop used for CS courses.

**Source:** `thread_laptop_specs.txt`

### Relevance Cutoff

I used a relevance cutoff of **0.65**. I chose this value by comparing the best retrieval distances for five questions that the corpus should answer with five questions that are outside the scope of the corpus.

**In-scope questions — best distances:**

- Bike/winter conditions: `0.521`
- Summer internship timing: `0.239`
- Laptop RAM: `0.200`
- Meal plan tier: `0.322`
- Professor email response time: `0.429`

**Out-of-scope questions — best distances:**

- Capital of Mongolia: `0.939`
- Changing oil in a diesel engine: `0.930`
- 1994 World Cup winner: `0.952`
- Recommended ibuprofen dosage: `0.782`
- Writing a for loop in Rust: `0.871`

The highest distance among the in-scope questions was `0.521`, while the lowest distance among the out-of-scope questions was `0.782`. I chose `0.65` because it falls comfortably between those two groups.

With this cutoff, all five in-scope questions passed the relevance gate and produced grounded answers. All five out-of-scope questions were rejected with:

`I don't have enough information about that.`

The rejected questions made zero model calls because the relevance gate stopped them before answer generation.

## What This Does

The Unofficial Guide is a retrieval-augmented question-answering system built around a corpus of student advice threads. It loads and chunks the advice documents, creates embeddings, retrieves the most relevant chunks for a user's question, and generates an answer grounded in those documents. The system can answer questions about topics such as internships, laptops, meal plans, commuting, and communicating with professors. A relevance gate prevents the system from answering questions that are not covered by the corpus. 
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

     ## How I Used AI

I used AI as a development assistant while building and testing the project.

One specific moment was choosing a chunking strategy. I asked AI to help me reason about why fixed character splitting was a poor fit for the `advice_threads` corpus. It suggested preserving paragraph and reply boundaries instead of cutting text at arbitrary character positions. I implemented a paragraph-aware `split_documents` function and then tested it myself by printing five chunks. All five sampled chunks contained enough context to understand at least one piece of advice independently.

Another specific moment was tuning the relevance cutoff. I gave AI the retrieval distances from my five in-scope questions and five out-of-scope questions and asked it to help compare them. The highest in-scope distance was `0.521`, while the lowest out-of-scope distance was `0.782`. Based on that evidence, I changed the cutoff from `0.6` to `0.65` and tested it. All five in-scope questions were answered, while all five out-of-scope questions were correctly refused.
