# Writing tasks and topics

This guide covers how to add a practice topic or change the diagnostic so that the checks still pass and every problem stays fair. The checks enforce the most important rule (section 3), so if `npm test` passes, the problem is sound.

## 1. Where things live

| File | What's in it |
|---|---|
| `public/sql/practice-bank.js` | `TOPICS` (each with a lesson, quick checks, problems and follow-up links) and `UPCOMING` (planned topics shown on the page) |
| `public/sql/lab-bank.js` | The diagnostic: `SKILLS`, `TASKS` and `CONCEPTS` |
| `checks/probes.js` | Per task: wrong answers that must fail, and alternative correct answers that must pass |
| `public/sql/core.js` | The engine and the task workspace. You shouldn't need to touch it to add content. |

## 2. A task

Practice problems and diagnostic tasks have the same shape:

```js
{
  id: 'best-seller',                    // unique across the whole project
  title: 'Best-selling product in each category',
  level: 'Easy',                        // Easy | Medium | Hard (practice); any label (diagnostic)
  skills: ['rank'],                     // diagnostic only: keys of SKILLS
  prompt: `<p>…</p>`,                   // HTML. Name every output column, its order, and the row order.
  setup: `CREATE TABLE …; INSERT INTO …;`,  // schema and sample rows
  starter: `WITH totals AS (…)`,        // optional: prefilled query, for "write one step" problems
  ref: `WITH …`,                        // the reference solution; it defines the right answer
  ordered: true,                        // true when the prompt asks for a row order
  hidden: [                             // tests: SQL run on top of setup
    { name: 'Home gets its own product called Tee (product 6), which sells 6 units',
      skill: 'agg',                     // diagnostic only: the one skill this test checks
      sql: `INSERT INTO products VALUES (6,'Tee','Home'); INSERT INTO order_items VALUES (8,6,6);` }
  ],
  muts: [                               // see section 3.2
    ['one total per product, not per name', 'GROUP BY p.product_id, p.category, p.name', 'GROUP BY p.name']
  ],
  hints: ['the approach', 'the skeleton', 'the key line'],
  explain: 'Shown with the reference solution once the problem is solved.'
}
```

**Expected output is never typed by hand.** For the sample data, each hidden test and any custom data, the expected output comes from running `ref` on the same data. So a test is only the rows it adds (or removes: `DELETE` works too).

## 3. Every requirement in a prompt is backed by a test

If a prompt asks for something, at least one test must fail when an answer leaves it out. `checks/mutate.js` enforces this. It breaks the reference solution one requirement at a time and fails unless some test then returns a **wrong result**. An SQL error doesn't count, because an answer that errors isn't the mistake being tested.

### 3.1 Checked on every task automatically

- **Remove the final `ORDER BY`.** The requested order has to differ from the order the rows would come out in anyway.
- **Flip each `ORDER BY` key.** Every sort key, each tie-break included, needs data where it changes the answer.
- **Remove each `ROUND`.** At least one expected value needs more decimals than the rounding keeps.

### 3.2 Listed per task in `muts`

Each entry is `[label, find, replace]`: a text edit to `ref` that removes one requirement. `find` must appear in `ref`. Add one for:

- each filter, such as a date range
- each boundary: `>=` vs `>`, and both edges of every `CASE` bucket
- each tie-break inside a window's `ORDER BY`
- the choice between `DENSE_RANK`, `RANK` and `ROW_NUMBER`
- `DISTINCT`, each `PARTITION BY`, the join type, and what the query groups by

### 3.3 Tie-breaks you can't test

A chain of tie-breaks has to end in one of two ways: a key the data really tests, or a stated guarantee ("no two reps tie on both"). A tie-break that matches the order rows come out in anyway can't be tested, so don't ask for it. Two common cases:

- **Alphabetical by name** is only testable when the query groups by an id whose order differs from the names' order.
- **"The earlier month"** matches the natural order of a `GROUP BY` on month. Ask for the later one, or break the tie on a second measure.

## 4. Hidden tests

- **One idea per test.** A test exists to catch one specific mistake.
- **Its name is the concrete situation**: "Two different managers both named Uma (ids 4 and 9), each with one report", not "managers who share a name". The name is shown when the test fails, and it's the hint.
- **In the diagnostic, tag each test with the one skill it checks.** The report counts a failed test against that skill only.
- Good sources of tests: ties at every tie-break level; NULLs (especially in `NOT IN`); duplicate rows; two entities that share a name; rows outside a date range on both sides; month, year and leap-day boundaries; a group with one member; a group with no rows; ids that aren't in date order.

## 5. Wrong and alternative answers

In `checks/probes.js`, under the task's id:

```js
'best-seller': {
  good: { byNameAndCategory: `…another correct query…` },
  bad:  { byName: [`…a query that groups by name only…`, 'Home gets its own product called Tee'] }
}
```

- `bad`: the query, and the start of the name of the test that must catch it (`null` means any test). Add every real mistake you see: the point is that it keeps failing after you change the data.
- `good`: other correct approaches (a join instead of a window, MySQL date functions instead of SQLite ones). This stops the tests from accepting only one way of writing the answer.

## 6. A practice topic

- **`lesson`**: `intro` (HTML), `setup`, `question`, `sql` (a worked example written as CTEs), `stepNotes` (one line per CTE name, plus `''` for the final result) and `outro`. The page runs the example and shows each step's output under its note.
- **`checks`**: 3–5 quick multiple-choice questions that ask for a prediction. `answer` is the index of the right choice; `why` explains it.
- **`problems`**, from easiest to hardest:
  1. Easy: the technique on raw rows, with no aggregation.
  2. Easy: a `starter` with every step written except the new one.
  3. Medium: a full query, adding one new element per problem (a join, a `CASE` bucket, a second tie-break, a second window).
  4. Hard: combines the topic with an earlier one.
- **`followups`**: real problems to try afterwards, as `{ n, title, slug }` (the slug is the problem's URL name on LeetCode). Link to them; don't copy their text or data.
- **Hints** come in three steps: the approach, then the skeleton, then the key line.

## 7. Before you publish

```bash
npm test
```

```bash
npm run e2e
```

`npm test` runs `harness.js` (reference solutions, wrong and alternative answers), `mutate.js` (section 3) and `macros.js` (the engine). `npm run e2e` drives the pages in a headless browser. It needs Playwright's Chromium: run `npx playwright install chromium` once. Its checks name specific diagnostic tasks and the first practice topic, so update them if you change those.
