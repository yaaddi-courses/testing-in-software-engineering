# Arrange, Act, Assert

**Covers:** Arrange, Act, Assert, AAA

## A shape that makes tests readable {#overview}
A test is read far more often than it is written. When it fails six months later, someone who has never seen it must work out what it checks and why it broke. A consistent shape makes that quick. Nearly every good test follows the same three steps in the same order, and learning the pattern helps you both write clear tests and spot unclear ones.

## Arrange {#arrange}
Arrange is the first step: set up everything the test needs before running the code. That might mean creating objects, preparing input values, filling a test database or choosing the current date. Anything that makes the situation concrete belongs here.

Keep the set-up tied to the test's point. If the test is about a discount, the price and the percentage are the important data. Extra detail that the test does not need makes it harder to see what matters.

## Act {#act}
Act is the second step: run the code being tested. This is usually one line, such as calling a function or sending a request. If you need more than a line or two to act, the code under test may be doing too much, or the test may be checking several things at once.

## Assert {#assert}
Assert is the third step: check that the result is what you expect. It compares the actual outcome with the expected one and fails with a clear message if they differ. Prefer one logical check per test, so a failure points to one cause. A few assertions on the same result are fine. A mix of unrelated checks is not.

## AAA {#aaa}
AAA is the short name for the whole pattern: Arrange, Act, Assert. People say "this test is not AAA" when set-up, action and checks are mixed together, or when assertions come before the action. A blank line between the three steps, or a short comment above each, makes the shape visible at a glance.

## A worked example {#example}
A function applies a percentage discount, and a test checks it.

```python
def apply_discount(price, percent):
    return round(price * (1 - percent / 100), 2)


def test_discount_reduces_price():
    # Arrange
    price = 80.0
    percent = 25

    # Act
    result = apply_discount(price, percent)

    # Assert
    assert result == 60.0


test_discount_reduces_price()
print("passed")
# passed
```

How to read it:

1. Arrange creates the two input values. A reader sees at once that the test is about 25 percent off 80.
2. Act calls the function once and keeps the result in a named variable.
3. Assert compares the result with 60.0, which a reader can check by hand: a quarter of 80 is 20, and 80 minus 20 is 60.

Compare with a muddled version that calls the function inside the assertion, with the numbers hidden in the call: `assert apply_discount(80.0, 25) == 60.0`. It works, but when it fails you cannot tell which part was set-up and which was the action. For a small test that is acceptable. For a long one, the three labelled parts are much easier to follow.

Another benefit is naming. A good test name states the expected behavior, such as `test_discount_reduces_price`. With AAA, the name, the set-up and the check all tell the same story.

## A common mistake {#mistake}
Putting several Act and Assert pairs in one test, so that it checks a whole sequence. When it fails, the cause is unclear. Split it into separate tests, one per behavior. A second mistake is a large Arrange step shared by many tests, which hides what each one depends on.

## Key points {#key-points}
- Arrange sets up the data, Act runs the code and Assert checks the result.
- AAA is the usual name for this three-step shape.
- Keep Act to one call and have one logical check per test.
- A name that states the behavior completes a readable test.
