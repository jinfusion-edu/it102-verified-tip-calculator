# How this works, and why

---

## 1. The concept being tested

**JavaScript's type coercion, and why you cannot trust arithmetic you did not
check.**

The assignment says it outright: *"AI models frequently struggle with basic data
types in JavaScript, sometimes concatenating strings (e.g. `"50" + "10" = "5010"`)
instead of performing mathematical addition."*

That example is not hypothetical. Everything read from an `<input>` is a
**string**, including `<input type="number">`. The `type` attribute constrains
what the user can type; it does not change what `.value` gives you.

```js
var bill = billInput.value;        // "50"   ← a string
var tip  = tipInput.value;         // "10"   ← a string

bill * tip / 100    // 5   — works! * coerces strings to numbers
bill + tipAmount    // "505"  — BROKEN. + concatenates when either side is a string
```

This is the nastiest class of bug in the language, because `*`, `-` and `/`
coerce to number while `+` prefers string concatenation. The multiplication in
this calculator would work by accident, and the addition on the very next line
would silently produce nonsense. Nothing throws. The page just shows a wrong
number.

The fix is one call, and Step 2 of the assignment calls it "Crucial":

```js
var billValue = parseFloat(billInput.value);   // 50, a real number
```

---

## 2. Walking the control flow

```
user clicks #calcBtn
        │
        ▼
read .value from both inputs        → "43.52", "18"   (strings)
        │
        ▼
parseFloat both                     → 43.52, 18       (numbers)
        │
        ▼
guard non-finite (empty → NaN → 0)
        │
        ▼
calculateBillTotals(43.52, 18)
        │   tipAmount  = 43.52 * 18 / 100 = 7.8336
        │   grandTotal = 43.52 + 7.8336   = 51.3536
        │   round both to 2dp             → 7.83, 51.35
        ▼
returns { tipAmount: 7.83, grandTotal: 51.35 }
        │
        ▼
.toFixed(2) and prefix "$"          → "$7.83", "$51.35"
        │
        ▼
written into #tipResult and #totalResult
```

### Why the guard is in the listener, not the function

The prompt says the function must contain *no* event listeners or HTML
manipulation. It says nothing about validation — but the function's job is
arithmetic, and mixing "what should happen when the field is blank" into it
would blur the boundary the assignment is teaching.

So the listener, which is the part that knows about a messy user interface,
normalises the input before handing clean numbers to the function:

```js
if (!isFinite(billValue)) {
  billValue = 0;
}
```

`isFinite` rejects `NaN`, `Infinity` and `-Infinity` in one check. Without it,
an empty bill field yields `NaN`, and `NaN` is contagious — every subsequent
operation produces `NaN`, and the user sees `$NaN`.

---

## 3. Rounding: number vs string, and when to do it

### Why the function returns numbers

`toFixed(2)` is the obvious way to "round to exactly two decimal places", but it
returns a **string**:

```js
(15).toFixed(2)        // "15.00"  ← a string
typeof (15).toFixed(2) // "string"
```

If the function returned that, the object would hold text. Any caller wanting to
do further arithmetic — add tax, split the bill between four people — would hit
the same string-concatenation bug all over again.

So the function rounds numerically:

```js
Math.round(tipAmount * 100) / 100      // 7.8336 → 783.36 → 783 → 7.83
```

Multiply by 100, round to a whole number, divide back. The result is a
**number** that happens to have at most two decimals.

The display layer then formats it:

```js
tipResultEl.textContent = "$" + totals.tipAmount.toFixed(2);
```

`toFixed(2)` is needed here because the number `15` must *display* as `15.00`.
Rounding and formatting are two different jobs, done in two different places.

### Why rounding happens at the end

There are two defensible strategies:

- **Round-at-end:** compute the exact tip, add it to the bill, then round both.
- **Round-then-add:** round the tip first, then add the rounded tip.

For the assignment's Test 2 they agree:

```
round-at-end:   43.52 + 7.8336 = 51.3536 → 51.35
round-then-add: 43.52 + 7.83   = 51.35   → 51.35
```

They agree on all three supplied test rows, so the professor's table does not
distinguish them. This code uses **round-at-end** — both returned values derive
from the unrounded `tipAmount` — because it keeps full precision as long as
possible, which is the standard accounting habit.

They do diverge. A worked example is in `AUDIT-NOTES.md`.

### The floating-point caveat

`toFixed` is not reliably "round half up", because decimal fractions cannot be
represented exactly in binary:

```js
(1.005).toFixed(2)   // "1.00", not "1.01"
```

`1.005` is actually stored as slightly *less* than 1.005. This affects the
calculator on any input landing exactly on a half-cent. Fixing it properly means
working in integer cents throughout, which is beyond the course's current scope.
It is documented rather than hidden.

---

## 4. Alternatives considered

**`Number()` instead of `parseFloat()`.** `Number("")` is `0` while
`parseFloat("")` is `NaN`; `Number("12abc")` is `NaN` while `parseFloat("12abc")`
is `12`. The assignment names `parseFloat` explicitly, so that ships. `Number()`
is arguably stricter and better here, and the following assignment (Event
Registration) uses it — because *there*, `Number("")` being `0` routes an empty
field into the "must be at least 1" error, which is the desired message.

**`toFixed(2)` inside the function.** Simpler, one call site instead of two.
Rejected: it returns strings, and the assignment asks for an object of *values*.

**Validating negatives in the function.** A negative bill currently computes
literally — a −$10 bill produces a −$1.50 tip. Rejected because the prompt
forbids anything but arithmetic, and defensive validation is the explicit subject
of a later assignment.

**`Intl.NumberFormat` for currency.** The correct production answer — it handles
locales, currency symbols and grouping. Well outside course scope, and `"$" +
toFixed(2)` is what the expected output shows.
