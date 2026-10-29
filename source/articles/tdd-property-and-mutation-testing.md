# TDD, property-based testing and mutation testing

**Covers:** TDD, property-based testing, mutation testing, mutant

## Three ways to get more from tests {#overview}
Writing tests is not the end of the story. You can write them in a different order to improve your design, you can let the computer invent inputs that you would never think of, and you can test the tests themselves. These three techniques address different weaknesses: designing before coding, relying on a few hand-picked examples and trusting a suite that may not catch anything.

## TDD {#tdd}
TDD stands for test-driven development. You write code in short loops of three steps, known as red, green, refactor:

1. **Red:** write a small test for behavior that does not exist yet, and watch it fail.
2. **Green:** write just enough code to make it pass.
3. **Refactor:** clean up the code and the test, keeping everything green.

Watching the test fail first proves that it can fail, so it is a real check. Writing it first makes you think about how the code will be used. TDD works best for logic with clear rules and less well for exploratory work, where you do not yet know what you want. It is a tool, not a religion.

## property-based testing {#property-based-testing}
Property-based testing checks a general rule across many generated inputs, instead of a few fixed examples. Instead of "sorting [3, 1, 2] gives [1, 2, 3]", you state a property: "for any list, the sorted result has the same length, and sorting it again changes nothing". A tool produces hundreds of random lists, including odd ones, and checks the property for each.

When it finds a failing input it usually shrinks it to the smallest example that still fails, which makes the bug easy to understand. It is strong at finding the edge cases that you did not think of. Libraries such as Hypothesis for Python provide the tools.

## mutation testing {#mutation-testing}
Mutation testing checks the quality of your tests by deliberately breaking the code in small ways and seeing whether the tests notice. A tool creates many slightly altered copies of your program, such as changing a plus to a minus or a "greater than or equal" to "greater than", and runs your suite against each one.

If the tests fail, the change was caught, and that is good. If they still pass, the change went unnoticed, which shows a gap in the tests. The share of changes caught is the mutation score. It is slow, so teams run it on the most important code, not everything.

## mutant {#mutant}
A mutant is one of those deliberately broken copies. A mutant is "killed" when at least one test fails because of it, and it "survives" when every test still passes. A surviving mutant is a finding: either a test is missing, or the change does not alter the behavior at all.

## A worked example {#example}
Here is a rule, a mutant of it, and two sets of tests, plus a small property check.

```python
import random


def is_adult(age):
    return age >= 18


def mutant(age):
    return age > 18


def weak_test(fn):
    return fn(10) is False and fn(30) is True


def strong_test(fn):
    return weak_test(fn) and fn(18) is True


print(weak_test(is_adult), weak_test(mutant))
print(strong_test(is_adult), strong_test(mutant))
# True True
# True False

random.seed(1)
for _ in range(200):
    items = [random.randint(-50, 50) for _ in range(random.randint(0, 10))]
    assert len(sorted(items)) == len(items)
    assert sorted(sorted(items)) == sorted(items)
print("200 random lists checked")
# 200 random lists checked
```

How to read it:

1. The mutant changes `>=` to `>`, which breaks the rule for exactly 18-year-olds.
2. The weak test passes for both the real code and the mutant, so the mutant survives. The tests never tried age 18.
3. The strong test adds a boundary check at 18. It passes for the real code and fails for the mutant, so the mutant is killed.
4. The last part states two properties of sorting and checks them on 200 random lists. No hand-written example was needed.

Mutation testing found the missing boundary test automatically. TDD would have encouraged writing that test first.

## A common mistake {#mistake}
Treating a high mutation score or a perfect TDD record as proof of correctness. They improve the tests, but they cannot check a requirement that was misunderstood. A second mistake is applying property-based tests with properties that are too weak, such as "the function returns something". Choose rules that really say what the code must do.

## Key points {#key-points}
- TDD means write a failing test, make it pass, then clean up.
- Property-based testing checks a rule on many generated inputs and shrinks failures.
- Mutation testing breaks the code on purpose to see if the tests notice.
- A surviving mutant points to a missing or weak test.
