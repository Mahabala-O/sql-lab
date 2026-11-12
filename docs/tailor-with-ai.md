# Tailor it to your gaps with an AI assistant

The diagnostic tells you where your gaps are. This guide turns that into a practice topic built for you, with Claude or another AI assistant doing most of the writing and the checks catching its mistakes.

## 1. Take the diagnostic

Open `sql-lab.html` (on your deployed copy, or locally) and work through the tasks and concept checks. Give each task 10–15 minutes before using a hint or the solution. Hints are recorded, which is the point: an honest map is more useful than a high score.

On the Report page, click **Copy results**. The summary includes your final query for each task, which hidden tests caught you, and which concept checks you missed. The queries matter most: they show *how* you think about a problem, not just whether you got it.

## 2. Ask for an evaluation

Paste this into the assistant, followed by your results:

```text
I took a SQL diagnostic. Below are my results, including my final query for
each task and the hidden tests that caught me. Read my actual queries, not just
the scores, and tell me:
1. What I'm solid on, with evidence from my queries.
2. My gaps, most important first. For each one, quote the query or test that
   shows it, and name the underlying concept (not just the task).
3. Which gap to work on first, and why.

[paste results here]
```

Check the answer against your own sense of what was hard. The report's verdicts are a rough guide; the queries are the evidence.

## 3. Ask for a practice topic

Give the assistant the rules and the format, then ask for one topic. Attach or paste `docs/writing-tasks.md` and the first topic in `public/sql/practice-bank.js` as an example.

```text
Write one new practice topic for this project, on: [the gap from step 2].

Follow docs/writing-tasks.md exactly, and use the existing topic in
practice-bank.js as the model for the format and tone. In particular:
- A lesson with a worked example written as CTEs, and a note for each step.
- 3–5 quick checks that ask me to predict an output.
- 6–8 problems from Easy to Hard. Problem 2 should give me a starter query with
  only the new step left to write.
- MySQL syntax in the reference solutions.
- Every requirement in each prompt needs a hidden test that catches its
  absence, and a `muts` entry that removes it.
- In checks/probes.js, add for each problem at least one wrong answer based on
  the mistakes in my diagnostic queries, and one alternative correct answer.

Context on my mistakes: [paste the relevant part of the evaluation]
```

## 4. Add it and let the checks review it

1. Add the topic to `TOPICS` in `public/sql/practice-bank.js`, and the answers to `checks/probes.js`.
2. Run the checks:

   ```bash
   npm test
   ```

3. When something fails, paste the output back to the assistant and ask it to fix the task, not the check. The usual failures:
   - `GAP … is not checked by any test`: a requirement with no test behind it. Add data where that requirement changes the answer, or remove it from the prompt.
   - `reference fails …`: the reference solution is wrong on one of its own tests.
   - `wrong answer … not caught`: a mistake the tests let through. Add a hidden test for it.
   - `correct answer … rejected`: the prompt is ambiguous, or the data has a tie it doesn't settle.
4. Open `public/sql-practice.html` and solve one or two problems yourself before trusting the rest.

## 5. Repeat

Work through the topic. When you can solve its Medium problems within the target time (15 minutes) without hints, take the diagnostic again: reset it on the Report page first. Then pick the next gap.
