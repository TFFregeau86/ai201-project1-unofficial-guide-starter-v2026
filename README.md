The Unofficial Guide

Name: T. Fregeau
Corpus: campus_life

Week 1
What This Does

The Unofficial Guide is a retrieval-based question-answering system built using the campus_life corpus. It searches short student posts about topics such as courses, housing, dining, campus services, and administrative policies. When a user asks a question, the system retrieves relevant chunks from the corpus and uses them to generate an answer grounded in those documents. It also names its sources and refuses to answer when the retrieved information is not relevant enough.

Chunking Strategy

Maximum chunk size: 400 characters
Overlap: 0 characters
Function: chunker.py::split_documents

I chose a paragraph-based chunking strategy because the campus_life corpus contains short student posts where useful information is usually contained in a sentence or short paragraph. The starter used fixed-size character windows with overlap, which did not pay attention to paragraph boundaries and could split useful information in the middle of a thought.

My split_documents function separates each document into paragraphs and combines neighboring paragraphs as long as the combined text stays within the 400-character limit. This keeps related paragraphs together while avoiding arbitrary cuts in the middle of paragraphs. I used no overlap because the chunks are created at paragraph boundaries, and the documents are already short enough that repeating text between chunks was not necessary.

After changing the chunker, the corpus produced 100 chunks from 88 documents, averaging 278 characters. The shortest chunk was 94 characters and the longest was 400 characters.

Sample Chunks
Chunk 1

Source: admin_add_drop_deadline.txt#0
Produced by: chunker.py::split_documents

On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

Chunk 2

Source: course_cs_210.txt#0
Produced by: chunker.py::split_documents

CS 210 Data Structures

I'm a junior and I've done this twice now. Format is lecture with weekly labs; slides go up after class, not before. Assessment: two midterms and a final, all drawn from lecture material rather than the textbook. Midterms are curved, the final is not.

Expect 8 to 10 hours a week outside class.

Chunk 3

Source: course_math_220_workload.txt#0
Produced by: chunker.py::split_documents

Workload for MATH 220 Linear Algebra

People keep asking so: 6 to 8 hours a week, almost all of it on problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.

Chunk 4

Source: dining_the_ridgeway_cafe_followup.txt#0
Produced by: chunker.py::split_documents

Re: The Ridgeway Café

Adding to what people have said about The Ridgeway Café. The wait figure of 10 to 15 minutes at 12:30 matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: seating is tight; about 40 seats for a building of 900. Nobody tells you this at orientation.

Chunk 5

Source: housing_morrow_house.txt#0
Produced by: chunker.py::split_documents

Morrow House — what it's actually like

Just finished a year in this building. Built 1954, partially renovated 2008. Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.

Sample Answer

Question: What happens if I drop a course after week two?

Answer:

If you drop a course after week two, it shows as a W on your transcript (admin_add_drop_deadline.txt).

Sources retrieved: admin_add_drop_deadline.txt, admin_declaring_a_major.txt, admin_grade_appeals.txt, admin_pass_fail_option.txt, admin_withdrawal_deadline.txt

Source: admin_add_drop_deadline.txt

Relevance Cutoff

Relevance cutoff: 0.6

I compared the best retrieval distances for my five in-corpus questions with the best distances for the five out-of-scope questions. The in-corpus questions had best distances between 0.2911 and 0.4610, while the out-of-scope questions had best distances between 0.8246 and 0.9340. There is a clear gap between the two groups, so the existing 0.6 cutoff falls between them.

Question	In corpus?	Best distance
What happens if I drop a course after week two?	Yes	0.3445
When should students book their adviser before registration?	Yes	0.4270
Are the CS 210 midterms curved?	Yes	0.4608
What time should students go to The Atrium to avoid a long wait?	Yes	0.2911
Does Calder Annexe have a building-wide noise problem?	Yes	0.3541
What is the capital of Mongolia?	No	0.8246
How do I change the oil in a diesel engine?	No	0.9340
Who won the 1994 World Cup?	No	0.8859
What is the recommended dosage of ibuprofen for a headache?	No	0.8442
How do I write a for loop in Rust?	No	0.8907

The lowest out-of-scope distance was 0.8246, while the highest in-corpus distance was 0.4608. This leaves a gap of more than 0.36, so 0.6 provides a substantial margin between relevant and unrelated questions.

How I Used AI

1. I used ChatGPT to help me understand how to approach the chunking strategy after examining the short campus_life documents. I considered a paragraph-based approach instead of blindly using fixed character windows. I then implemented and tested the strategy myself, using a 400-character maximum and no overlap, and inspected the chunks produced by my code.

2. I used ChatGPT to help me check whether my acceptance criteria were specific enough to be tested. I kept the final criteria based on my own decisions about the corpus and changed the explanations so that each target had a reason connected to my documents or pipeline.

Week 2
Run Log — Before

I ran python run_eval.py --label before before building scorer.py. The evaluation reported that no scorer was present, so criteria 1, 2, 4, and 5 were not automatically scored yet. The retrieval distances were recorded for all five in-corpus questions, and the relevance gate was evaluated for all five out-of-scope questions.

Criterion	Target	Run 1	Run 2	Run 3	Verdict
1. Retrieved chunk contains the answer	4 of 5	Not scored	Not scored	Not scored	Pending scorer
2. Every answer names a source	5 of 5	Not scored	Not scored	Not scored	Pending scorer
3. Gate stops out-of-corpus questions	4 of 5	5 of 5	5 of 5	5 of 5	MET
4. Chunks contain complete thoughts	4 of 5	Not scored	Not scored	Not scored	Pending scorer
5. Answers contain expected information	4 of 5	Not scored	Not scored	Not scored	Pending scorer
Criterion 3 evidence

The five out-of-scope questions were all rejected by the relevance gate:

Question	Best distance	Result
What is the capital of Mongolia?	0.8246	Refused
How do I change the oil in a diesel engine?	0.9340	Refused
Who won the 1994 World Cup?	0.8859	Refused
What is the recommended dosage of ibuprofen for a headache?	0.8442	Refused
How do I write a for loop in Rust?	0.8907	Refused

The gate therefore refused 5 of 5 out-of-scope questions, exceeding the target of 4 of 5.

Run file: results/run_2026-09-14_2250_before.md

Command used: python run_eval.py --label before

Verdicts
#	Criterion	Verdict	How I decided
1	Retrieved chunk contains the answer	Pending	The unscored evaluation recorded retrieval distances but did not yet determine whether the retrieved chunks contained the complete answer.
2	Every answer names a source	Pending	scorer.py was not present, so this criterion was not automatically scored in the before run.
3	Gate stops out-of-corpus questions	MET	The gate refused all 5 out-of-scope questions, which is better than the target of 4 of 5.
4	Chunks contain complete thoughts	Pending	This requires inspecting the sampled chunks against the criterion; the unscored evaluation did not provide a numerical verdict.
5	Answers contain expected information	Pending	The evaluation did not yet have a scorer to compare answers with the expects phrases.
Diagnoses

The before evaluation shows one clear result: the relevance gate is working well for the five out-of-scope questions. All five had distances above the 0.6 cutoff and were refused.

The remaining four criteria cannot be honestly diagnosed from the unscored run alone. Their measurements require the Week 2 scorer and inspection of the retrieved chunks and generated answers. I will use those results to determine whether any failures come from loading, chunking, embedding, retrieval, or generation.

The current chunking strategy itself produced 100 chunks from 88 documents and keeps paragraphs intact, so one thing I will check is whether any answer is spread across multiple chunks or whether the retrieval ranking places the answer-containing chunk below the top result.

The Improvement

What I changed:

I changed the starter's fixed-size character-window chunker to chunker.py::split_documents, which groups complete paragraphs up to 400 characters with no overlap.

Why I picked it:

The starter chunker could split text at arbitrary character positions. The campus_life corpus consists mostly of short posts and paragraphs, so preserving paragraph boundaries should make each retrieved chunk more self-contained and easier for the model to use.

Run Log — After

This section will be completed after the Week 2 improvement run and scorer results are available.

Criterion	Target	Run 1	Run 2	Run 3	Verdict
1. Retrieved chunk contains the answer	4 of 5	Pending	Pending	Pending	Pending
2. Every answer names a source	5 of 5	Pending	Pending	Pending	Pending
3. Gate stops out-of-corpus questions	4 of 5	Pending	Pending	Pending	Pending
4. Chunks contain complete thoughts	4 of 5	Pending	Pending	Pending	Pending
5. Answers contain expected information	4 of 5	Pending	Pending	Pending	Pending
Did It Help?

The chunking change produced 100 chunks instead of the starter's 88 chunks and kept the sampled chunks as complete paragraphs. However, I will use the Week 2 before/after evaluation results to determine whether this structural improvement actually improved retrieval or answer quality.

What's Still Broken

At this point, the only criterion with a completed measurement is criterion 3, and it is met at 5 of 5.

Criteria 1, 2, 4, and 5 still need their scorer-based measurements before I can identify which ones remain broken. I will not lower their original targets based on missing results.

What I'd Do Differently

I would keep the original targets because they were specific and measurable before seeing the results. The main thing I would improve is the measurement plan: I would make sure the scorer is available before running the full Week 2 evaluation so that retrieval, source attribution, chunk completeness, and expected-answer content can all be measured consistently across the before and after runs.

The relevance-gate criterion was particularly useful because it produced a direct measurement: all five clearly unrelated questions were rejected, and their distances were substantially higher than the in-corpus questions. That supports the original decision to use a 0.6 cutoff.