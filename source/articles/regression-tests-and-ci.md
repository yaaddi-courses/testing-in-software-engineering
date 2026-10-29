# Regression tests and the CI pipeline

**Covers:** regression test, CI pipeline

## Fixed once, fixed for good {#overview}
Every team knows the feeling of fixing a bug and finding it back a month later. Software changes constantly, and a change in one place can break something that used to work. Two practices deal with this together. The first is a test for every old failure, run again after every change. The second is an automatic system that does the running, so that nobody has to remember.

## regression test {#regression-test}
The word regression means going backward. A regression test is a test you keep and run again after every change, to catch something that used to work and has quietly broken again. The most valuable ones are written when a bug is found: first write a test that reproduces the bug and fails, then fix the bug, then keep the test for good.

That habit turns each bug into a permanent guard. Over time, the suite becomes a record of everything that has ever gone wrong in this particular system, which makes the same mistake unlikely to return.

A regression test should be small and specific, with a name that says what it protects, such as `test_average_of_empty_list`. A comment with the bug's ticket number helps the next person understand why it exists.

## CI pipeline {#ci-pipeline}
Who runs all these tests? A machine does. A CI pipeline is the automation that runs your tests every time someone pushes a change to the codebase, and blocks a bad merge if anything fails. CI stands for continuous integration. The pipeline checks out the code, builds it, runs the tests and reports a pass or fail on the change, usually within minutes.

This is what makes regression tests useful. A suite that has to be run by hand is run less and less as people get busy. A pipeline runs it every time, with no exceptions and no reminders.

Typical stages are: build, fast unit tests first, then integration tests, then slower checks such as linting and security scans. Putting the fastest checks first gives quick feedback. A failed build is fixed straight away, before anything else is merged.

## A worked example {#example}
A bug report says that the average of an empty list crashes the report page. The developer follows the regression routine.

```python
def average(values):
    # Bug fix: an empty list used to crash with ZeroDivisionError
    return sum(values) / len(values) if values else 0


def test_average_of_empty_list_regression():
    assert average([]) == 0


def test_average_normal():
    assert average([2, 4, 6]) == 4


for test in (test_average_of_empty_list_regression, test_average_normal):
    test()
print("2 tests passed")
# 2 tests passed
```

Let's go through the steps:

1. The developer first writes `test_average_of_empty_list_regression`. Against the old code it fails with a `ZeroDivisionError`, which proves that it reproduces the bug.
2. The fix adds the `if values else 0` guard. The test passes.
3. The existing test `test_average_normal` still passes, which shows that the fix did not break the normal case.
4. The change is pushed. The CI pipeline runs both tests, together with the other 800 tests, in five minutes and reports green.
5. Six months later, a refactoring removes the guard by mistake. The pipeline fails within minutes, with the name of the test that explains exactly what broke and why.

Without the pipeline, that refactoring might have gone to users, and the same crash would have returned as a surprise.

## A common mistake {#mistake}
A common slip is fixing a bug without writing the test, "because it was a one-line fix". These are the bugs that return.

A second slip is a pipeline that people ignore when it fails, or a suite so slow that they stop waiting for it. Keep the feedback fast and treat red as a stop sign.

## Key points {#key-points}
- A regression test guards against an old failure coming back. Write one with every bug fix.
- Write the test first and watch it fail, then fix the bug.
- A CI pipeline runs the tests on every push and blocks a bad merge.
- Put fast checks first and fix a broken build immediately.
