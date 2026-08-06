# Test cases

The assignment supplies a three-row verification table (Step 3) and separately
asks for test cases in the video. The professor's three are the normal cases;
three edge cases were authored to complete the set.

> All rows below were **executed** by `node ../../tools/run_tests.js`, which
> loads the real `<script>` block from `tip-calculator.html`, fires the actual
> click handler, and reads the actual `#tipResult` / `#totalResult` elements.
> The "Actual" column is harness output, not prediction.

## Step 3 — the professor's verification sandbox

| Test Case | Inputs | Expected Output | Actual Output | Status |
|---|---|---|---|---|
| Test 1 | Bill `$100.00`, Tip `15%` | Tip `$15.00` / Total `$115.00` | Tip `$15.00` / Total `$115.00` | ✅ **PASS** |
| Test 2 | Bill `$43.52`, Tip `18%` | Tip `$7.83` / Total `$51.35` | Tip `$7.83` / Total `$51.35` | ✅ **PASS** |
| Test 3 | Bill `$0.00`, Tip `20%` | Tip `$0.00` / Total `$0.00` | Tip `$0.00` / Total `$0.00` | ✅ **PASS** |

Test 2 is the one that matters: `43.52 × 0.18 = 7.8336`, which must round to
`7.83`, and `43.52 + 7.8336 = 51.3536`, which must round to `51.35`. A naive
implementation that concatenated strings would show `43.527.8336`.

## Edge cases

### E1 — Bill field left empty
- **Input:** Bill *(empty)*, Tip `15`
- **Expected:** `$0.00` / `$0.00` — **not** `$NaN`
- **Actual:** ✅ **PASS** — `["$0.00","$0.00"]`
- **Why:** `parseFloat("")` is `NaN`, and `NaN` propagates through every
  subsequent operation. Without the `isFinite` guard the user would see `$NaN`.

### E2 — Tip percentage cleared
- **Input:** Bill `50`, Tip *(empty)*
- **Expected:** `$0.00` tip, `$50.00` total — a 0% tip, not a broken total
- **Actual:** ✅ **PASS** — `["$0.00","$50.00"]`

### E3 — Very large bill
- **Input:** Bill `1000000000`, Tip `15`
- **Expected:** `$150000000.00` / `$1150000000.00` — no overflow, no exponential
  notation
- **Actual:** ✅ **PASS**
- **Note:** readable but ugly — there are no thousands separators. Beyond
  `Number.MAX_SAFE_INTEGER` precision would degrade silently; no guard exists.

## Further executed assertions

| Check | Result |
|---|---|
| `typeof totals.tipAmount === "number"` | ✅ PASS |
| `typeof totals.grandTotal === "number"` | ✅ PASS |
| Negative tip percent computes literally (`100 @ -10%` → `$-10.00` / `$90.00`) | ✅ PASS |

That last row is **not** a bug being reported as a pass — it is confirmation of
*deliberately unhandled* behaviour. The prompt forbids validation inside the
function, and defensive input handling is the subject of a later assignment. It
is recorded here so the behaviour is known rather than discovered.

## Explicitly NOT tested

- Any visual rendering — no browser was opened
- The `index.html` → `tip-calculator.html` redirect against a live deployment
- Half-cent rounding (`1.005` → `"1.00"`); documented in `AUDIT-NOTES.md` as a
  known floating-point limitation, not exercised here
- What `<input type="number">` puts in `.value` for malformed keystrokes
  like `1.2.3`
