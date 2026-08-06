# The Verified Tip Calculator

IT102 · Introduction to Programming · Seattle Colleges

A tip calculator whose arithmetic has been verified against the assignment's
test table rather than assumed.

## What it does

Takes a bill amount and a tip percentage, and on click displays the tip and the
grand total, each formatted to two decimal places.

The calculation lives in `calculateBillTotals(bill, tipPercentage)`, which is
pure arithmetic — it returns an object of numbers and never touches the page.

## Files

| File | Role |
|---|---|
| `tip-calculator.html` | **The assignment deliverable.** Starter markup + the `<script>` block. |
| `index.html` | A one-line redirect to the above, so the live URL works. |

The assignment specifies the filename `tip-calculator.html`. GitHub Pages serves
`index.html` by default, so visiting the bare live URL would 404 without the
redirect. The graded file keeps its required name.

## How to run it

```bash
git clone https://github.com/jinfusion-edu/it102-verified-tip-calculator.git
cd it102-verified-tip-calculator
```

Open `tip-calculator.html` in a browser. No dependencies, no build step.

## Expected output

A card headed **Tip Calculator** with a bill field, a tip-percentage field
pre-filled with `15`, a green **Calculate Total** button, and a results box.

| Bill | Tip % | Tip Amount | Total Bill |
|---|---|---|---|
| 100.00 | 15 | `$15.00` | `$115.00` |
| 43.52 | 18 | `$7.83` | `$51.35` |
| 0.00 | 20 | `$0.00` | `$0.00` |

## Live URL

https://edu.jinfusion.dev/it102-verified-tip-calculator/

## AI collaboration — tool and prompt

**Tool used:** Claude (Anthropic), via Claude Code.

The assignment specifies the prompt exactly:

> Act as a JavaScript developer. Write a function called "calculateBillTotals".
> It should accept two parameters: "bill" and "tipPercentage".
>
> The function needs to calculate:
> 1. The tip amount (bill multiplied by tipPercentage divided by 100).
> 2. The grand total (bill plus the tip amount).
>
> The function must return an object containing both values, rounded to exactly
> two decimal places.
> Return only the JavaScript code. Do not include any event listeners or HTML
> manipulation code.

The event listener was written by hand, per Step 2.

### What I corrected after reviewing the output

- **"Rounded to two decimal places" is ambiguous between a number and a string.**
  `toFixed(2)` returns a *string*, which would make the returned object hold text
  rather than values. The function returns **numbers**; the `$0.00` formatting
  happens in the display layer.
- **Empty input produces `NaN`.** `parseFloat("")` is `NaN`, and `NaN` propagates
  silently into `"$NaN"` on screen. Guarded in the listener, not in the function,
  so the function stays pure as the prompt demanded.

Full reasoning in `AUDIT-NOTES.md`.

## Verification

```bash
node ../../tools/run_tests.js
```

Loads the real `<script>` block, drives the actual click handler, reads the
actual result elements. **All three of the professor's test cases pass**, plus
six further assertions. See `TEST-CASES.md`.
