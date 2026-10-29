# Coverage, boundaries and the happy path

**Covers:** code coverage, boundary case, happy path

## A number that is easy to misread {#overview}
Teams like measurable goals, and test coverage gives one: a percentage on a dashboard. But the number says less than it seems to. This article explains what it really measures, which kinds of inputs are most likely to hide bugs, and why testing only the normal case leaves large holes.

## code coverage {#code-coverage}
Code coverage is the percentage of your code's lines that run at least once while your tests execute. A tool marks each line as run or not run, and the report shows the share that ran.

What it proves is narrow. A covered line was executed. It does not prove that anything checked the result. A test that calls a function and asserts nothing gives full coverage and zero protection. Coverage is best used in one direction: low coverage tells you where nothing is tested, and high coverage does not tell you that the tests are good.

Other forms exist. Branch coverage counts whether both outcomes of each decision (the "if" and the "else") ran, which is stricter than line coverage.

## happy path {#happy-path}
The happy path is the normal, expected flow in which everything works and no errors occur. A shopper enters a valid card, the payment succeeds and the order is confirmed. It is the easiest to test and the first test most people write.

It is not enough. Most real-world failures come from the other paths: bad input, empty data, timeouts. A test suite that covers only the happy path can reach very high coverage numbers while the program remains fragile.

## boundary case {#boundary-case}
A boundary case is an input at the edge of what the code handles: zero, one, the maximum, an empty list, a negative number, the first or last item. Bugs cluster here, often because of off-by-one errors where a "less than" should have been "less than or equal".

A useful habit is to test at the edge and on both sides of it. If a discount starts at 100 dollars, test 99, 100 and 101. If a function takes a list, test an empty list, a list with one item and a list with many.

## A worked example {#example}
A function returns the first word of a text. Its happy path is a sentence. The boundaries are the empty text, a text with only spaces and a single word.

```python
def first_word(text):
    parts = text.split()
    return parts[0] if parts else ""


cases = {
    "hello world": "hello",  # happy path
    "": "",  # boundary: empty
    "   ": "",  # boundary: only spaces
    "one": "one",  # boundary: single word
}

for text, expected in cases.items():
    assert first_word(text) == expected
print(len(cases), "cases passed")
# 4 cases passed
```

How to read it:

1. The first case is the happy path, and it alone would give 100 percent line coverage if the code had no `if`.
2. The empty text and the spaces-only text are boundary cases. A version of the function that used `text.split()[0]` directly would crash with an index error on both.
3. The single word checks the other end: a list with exactly one item.

Now think about coverage. With only the happy path, the line `return ... if parts else ""` ran, so line coverage reports 100 percent. Yet the `else` side never ran. Branch coverage would show 50 percent for that decision. The boundary cases bring it to 100 percent and, more importantly, protect against real crashes.

## A common mistake {#mistake}
Chasing a coverage target such as 90 percent by writing tests that execute lines without checking anything. The number rises and the safety does not. Set coverage as a floor that flags untested code, and judge test quality by whether the tests would fail when the behavior breaks.

## Key points {#key-points}
- Code coverage shows which lines ran, not whether the results were checked.
- The happy path is the normal flow. It is necessary and not enough.
- Boundary cases, such as zero, empty and the maximum, are where bugs hide.
- Test at the edge and on both sides of it.
