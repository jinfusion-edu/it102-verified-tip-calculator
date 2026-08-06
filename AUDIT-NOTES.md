# Audit notes

Written to be attacked. Organised against the assignment's Steps 1–3, whose
Step 3 *is* the verification checklist.

---

## Against the assignment's steps

### ▸ Prerequisite — starter code reproduced exactly, as `tip-calculator.html`
**Yes, byte-for-byte.** Only `// YOUR CODE WILL GO HERE` was replaced.

**Deviation to declare:** an extra `index.html` was added containing a
`<meta http-equiv="refresh">` redirect to `tip-calculator.html`. Reason: the
assignment fixes the filename, but GitHub Pages serves `index.html` by default,
so the required "Live Website URL" would 404 on the bare repo URL. The graded
file is untouched and keeps its required name. If a grader counts files, there
are two HTML files, and this is why.

### ▸ Step 1 — the function
- Named `calculateBillTotals` ✅
- Parameters `bill`, `tipPercentage` ✅
- Tip = `bill * tipPercentage / 100` ✅
- Total = `bill + tip` ✅
- Returns an **object** with both values ✅
- Rounded to exactly two decimal places ✅ — as numbers, see below
- **No event listeners, no DOM manipulation inside it** ✅ (verified by reading;
  the function body references only its two parameters)

### ▸ Step 2 — implement and connect
- Function inside `<script>` ✅
- `'click'` listener on `#calcBtn` ✅
- `.value` captured from `#billAmount` and `#tipPercent` ✅
- **`parseFloat()` applied before the values reach the function** ✅ — the step
  the assignment marks "Crucial"
- Object unpacked; `#tipResult` and `#totalResult` updated ✅

### ▸ Step 3 — Verification Sandbox (the professor's table)

**All three executed, not eyeballed.** Harness output:

| Test | Inputs | Expected | Actual | Status |
|---|---|---|---|---|
| 1 | 100.00 @ 15% | $15.00 / $115.00 | `$15.00` / `$115.00` | ✅ PASS |
| 2 | 43.52 @ 18% | $7.83 / $51.35 | `$7.83` / `$51.35` | ✅ PASS |
| 3 | 0.00 @ 20% | $0.00 / $0.00 | `$0.00` / `$0.00` | ✅ PASS |

---

## Beyond the checklist

### The interpretation that matters: "rounded to exactly two decimal places"

Ambiguous between a **number** rounded to 2dp and a **string** formatted to 2dp.
`toFixed(2)` returns a string; `Math.round(x*100)/100` returns a number that
will not *display* as `15.00`.

Chosen: the function returns **numbers**, the display layer calls `.toFixed(2)`.
Rationale — the prompt asks for an object of *values*, and a caller doing further
arithmetic on a string would hit exactly the concatenation bug the assignment is
about.

**Counter-argument, stated fairly:** a grader reading `calculateBillTotals(100, 15)`
in a console sees `{ tipAmount: 15, grandTotal: 115 }`, not `15.00`. If they
expected the function itself to yield `"15.00"`, this reads as a miss. The
*screen* is correct either way.

### Rounding strategy, and where the two disagree

Both returned values are derived from the **unrounded** tip (round-at-end). The
alternative is to round the tip and then add it.

All three supplied test rows agree under both strategies, so the professor's
table does not discriminate between them. A case that does:

```
bill 0.05, tip 50%
  exact tip   = 0.025
  round-at-end:   tip → 0.03 ; total = 0.05 + 0.025 = 0.075 → 0.08
  round-then-add: tip → 0.03 ; total = 0.05 + 0.03  = 0.08  → 0.08
```

— still agree. A genuine divergence needs the rounded and unrounded totals to
straddle a half-cent, e.g. `bill 0.01, tip 25%`: exact tip `0.0025` → displays
`0.00`, total `0.0125` → `0.01`, so the receipt shows a $0.01 total with a $0.00
tip. **The displayed tip and total will not always add up.** That is inherent to
rounding two derived values independently, is true of most real receipts, and is
worth knowing.

### Floating point: a concrete failing input

`toFixed` is not reliably half-up:

```js
(1.005).toFixed(2)   // "1.00"  — not "1.01"
```

`1.005` is stored as marginally less than 1.005. So a bill and tip landing
exactly on a half-cent can round down where a human would round up. Correcting
this requires integer-cent arithmetic throughout, which is beyond the course's
scope. **Known, unfixed, deliberate.**

### What I executed vs. what I only reasoned about

**Executed** — `node ../../tools/run_tests.js`, 9 assertions, all passing: the
three professor cases, `typeof` checks on both returned values, and four edge
cases (empty bill, empty tip, negative tip percent, 1e9 bill).

**Reasoned about, NOT executed:**
- **Any visual rendering.** No browser opened. The result box, the green button,
  the flex `.result-line` layout — all unverified visually.
- **The `1.005` floating-point claim** — this is standard, documented JS
  behaviour, but I did not run it in this project's harness.
- **The redirect actually working on GitHub Pages.** `meta refresh` is
  well-supported; not confirmed against a live deployment.
- **`<input type="number">` browser behaviour** — e.g. what Chrome puts in
  `.value` when the user types `1.2.3` (it gives `""`). The empty-string path is
  tested; that specific browser interaction is not.

### Edge cases known to be unhandled

- **Negative bill or negative tip.** Computed literally: `-10` bill at 15%
  produces `$-1.50` / `$-11.50`. Not in the professor's table, and the prompt
  forbids validation inside the function. **Deliberately unhandled** — defensive
  input handling is the explicit subject of the Event Registration assignment.
- **Half-cent rounding**, above.
- **Very large numbers.** `1e9` was tested and formats fine, but beyond
  `Number.MAX_SAFE_INTEGER` precision degrades silently. No guard.
- **The tip field is pre-filled with `15`**, so it is never empty on load. The
  user can clear it; that path is tested and yields `$0.00`.
- **No thousands separators.** `$1150000000.00` is hard to read. `Intl.NumberFormat`
  would fix it and is out of scope.
- **Results are not cleared when inputs change** — stale figures remain visible
  until the button is pressed again. The assignment is click-driven, so this is
  per spec, but a user could misread an old total.

### Three places I would look first if this turned out to be wrong

1. **The `parseFloat` calls.** If a total ever appears as a concatenation
   (`"43.527.83"`) or the tip is right while the total is absurd, one of the two
   `parseFloat` calls was removed or applied after the arithmetic.
2. **The `isFinite` guards.** `$NaN` on screen means a guard was bypassed —
   most likely someone "simplified" `if (!isFinite(x)) x = 0;` away.
3. **`Math.round(x * 100) / 100` vs `toFixed`.** If the object suddenly holds
   strings, downstream `.toFixed(2)` still works (strings have no `toFixed` —
   it would throw `totals.tipAmount.toFixed is not a function`). That exact
   error message points straight here.

### What I would flag reviewing this as someone else's code

- `var` throughout — course-level choice, would be flagged anywhere else.
- The two `isFinite` guards are near-identical four-line blocks; a small helper
  would remove the duplication, at the cost of indirection a beginner reader
  must follow.
- `"$" + x.toFixed(2)` hardcodes both the currency symbol and its position.
  Fine for this assignment, wrong for anywhere that is not the US.
- The function returns `{ tipAmount, grandTotal }`, but the assignment text
  never names the object's keys. Any grader auto-checking for `.tip` and
  `.total` would fail. The names are readable and I would not change them, but
  the risk exists and nothing in the spec pins it down.

### Nothing found clean

Not clean. The half-cent rounding divergence and the number-vs-string
interpretation are both real, and the object key names are an unpinned
assumption that a strict automated grader could trip over.
