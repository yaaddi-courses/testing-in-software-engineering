# Browser tests, flaky tests and race conditions

**Covers:** browser-driven test, flaky test, race condition

## The most realistic tests are the hardest to keep stable {#overview}
A browser-driven test is as close as automation gets to a real user. That realism has a price: such tests are the likeliest to pass on one run and fail on the next. A test that cannot be trusted is worse than no test, because people learn to ignore failures. In this article we will look at this kind of test, at the problem it causes, and at one of its main causes.

## browser-driven test {#browser-driven-test}
Picture a robot that opens a real browser, clicks buttons, fills in forms and reads the page, exactly as a person would. That is a browser-driven test. Tools such as Playwright and Cypress make it possible. The test opens the site, types in a username, presses Log in and then checks that the welcome message appears.

These tests check the whole system, including the layout, the scripts that run in the browser and the server behind them. They are slow, since a real browser must start and load pages, and they break whenever the page's structure changes. Use them for a few critical journeys, such as signing up and paying, and not for every rule.

Here is good practice. Find elements by their role or label, such as "the button called Log in", and not by fragile details like the third `div`. Wait for a condition, such as "the message is visible", and not for a fixed number of seconds.

## flaky test {#flaky-test}
Have you ever seen a test fail, rerun it, and watch it pass with nothing changed? That is a flaky test: it sometimes passes and sometimes fails on exactly the same code. The usual causes are timing (the test checks before the page has finished loading), shared state (tests that depend on the leftovers of other tests), random data and slow or unreliable external services.

The damage is trust. Once a team knows that "it is probably only flaky", they rerun the build and ignore the red result, and a real failure slips through. Treat a flaky test as a bug. Fix its cause, or quarantine it, meaning remove it from the main build and track it, so it cannot teach people to ignore failures.

## race condition {#race-condition}
One of the most common causes of flakiness is a race condition, a bug in which the result depends on the exact timing of two things happening at once. For example, two parts of the program both read a value, change it and write it back. If their steps interleave in an unlucky order, one update overwrites the other.

Race conditions can sit in the program itself or in the test, when it assumes that something has finished before it really has. In both cases the code works whenever the timing happens to be lucky.

## A worked example {#example}
Here is a "lost update" with the steps written out in the unlucky order, so the result is the same every time.

```python
counter = 0


def read():
    return counter


def write(value):
    global counter
    counter = value


# Two workers each do "read, add 1, write", but their steps interleave
a = read()  # worker A reads 0
b = read()  # worker B reads 0
write(a + 1)  # A writes 1
write(b + 1)  # B writes 1, overwriting A's update
print(counter)
# 1
```

Let's read it:

1. Each worker should add one, so the correct answer after two workers is 2.
2. Both workers read the value 0 before either wrote. Each computes 0 + 1.
3. Both write 1. The second write erases the first worker's work, and the final answer is 1.

If the workers ran one after the other, the answer would be 2. In real programs the order depends on timing, so the result varies from run to run. A test of this code passes on a quiet machine and fails on a busy one. That is a flaky test caused by a race condition.

The fix in the program is to make read-and-write a single protected step, for example with a lock. The fix in a browser test is to wait for the actual condition, such as "the counter text shows 2", and not for two seconds.

## A common mistake {#mistake}
A common slip is adding a fixed delay, such as waiting three seconds, to quiet a flaky test. It hides the race on your machine and fails on a slower one, and it makes the whole suite slower. Wait for the condition you need, with a time limit.

A second slip is rerunning failed builds automatically and never looking at why.

## Key points {#key-points}
- A browser-driven test acts like a real user and is slow and fragile, so use it for key journeys only.
- A flaky test passes and fails on the same code. Fix it or quarantine it.
- A race condition is a bug whose result depends on timing.
- Wait for conditions, not for a fixed number of seconds.
