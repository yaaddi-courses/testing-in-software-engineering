# Test levels and the test pyramid

**Covers:** unit test, integration test, end-to-end test, E2E, UI, UIs, test pyramid

## Testing at three distances {#overview}
Automated tests do not all do the same job. Some look closely at one small piece of code. Some check that pieces fit together. Some drive the whole application like a user. Each level answers a different question, and each costs a different amount to write, run and maintain. Mixing them well is one of the most useful skills in testing, so let's go through the levels one by one.

## unit test {#unit-test}
A unit test checks one small piece of code, usually a single function, on its own. It does not touch a real database, the network or other parts of the system. Anything the code depends on is replaced by a plain stand-in.

Unit tests are fast, often a few milliseconds each, so thousands can run in seconds. They are precise: when one fails, you know which function is wrong. They cannot show that parts work together, because by design they look at one part alone.

## integration test {#integration-test}
The next level asks whether the parts fit together. An integration test checks that several parts of the system work together for real. Your code talks to an actual database, or two services call each other. This is where mistakes in the connections show up: a column with the wrong name, or a date format that one side writes and the other cannot read.

These tests are slower and take more set-up than unit tests, since a real database or service must be running. You write fewer of them and aim them at the places where parts meet.

## end-to-end test {#end-to-end-test}
At the top level, an end-to-end test drives the application the way a real user would, from the start of a flow to its end: open the page, log in, add an item, pay, see the confirmation. It checks the whole stack at once, from the screen down to the database.

These give the most realistic result and the least precise one. A failure could come from anywhere, and the tests are the slowest and most fragile of all.

## E2E {#e2e}
E2E is the common short form of end-to-end. You will see "the E2E suite" or "E2E tests" in team chat and in build logs. It means the same slow, realistic tests described above.

## UI {#ui}
UI means user interface: the screens, buttons and controls through which people use software. It matters in testing because end-to-end tests usually work through the UI, finding buttons and fields on the screen. The UI also changes more often than the logic underneath, so tests tied to its exact layout break easily.

## test pyramid {#test-pyramid}
How should you mix the levels? The test pyramid is a way to plan a test suite: many unit tests at the wide base, fewer integration tests in the middle and a small number of end-to-end tests at the narrow top. The shape follows the cost. The lower levels are cheap and fast, so you can have many. The higher levels are expensive and slow, so you keep only the most valuable ones.

The opposite shape, with mostly slow end-to-end tests, is called an ice-cream cone. It gives slow feedback, many flaky failures and expensive maintenance.

## A worked example {#example}
A team has 1,000 tests. Let's see how the run time depends on the mix.

```python
suites = {"unit": (700, 0.01), "integration": (200, 0.5), "end-to-end": (100, 20)}

for name, (count, seconds) in suites.items():
    print(name, count * seconds)
# unit 7.0
# integration 100.0
# end-to-end 2000
```

Let's read it:

1. 700 unit tests at 0.01 seconds each take 7 seconds in total.
2. 200 integration tests at half a second take 100 seconds.
3. 100 end-to-end tests at 20 seconds take 2,000 seconds.
4. The total is 7 + 100 + 2,000 = 2,107 seconds, about 35 minutes. The end-to-end tests are 10 percent of the tests but about 95 percent of the time.

Now imagine the team had written 700 more end-to-end tests instead of 700 unit tests. Those 700 tests alone would take 700 times 20, which is 14,000 seconds, almost four hours. A developer would stop running the tests before each change. The pyramid keeps the feedback fast.

## A common mistake {#mistake}
A common slip is testing everything through the screen because it feels closest to the user. It is slow, and the failures are hard to diagnose. Test each rule at the lowest level that can show it, and keep end-to-end tests for the few journeys that must never break.

## Key points {#key-points}
- A unit test checks one piece alone, an integration test checks pieces together and an end-to-end test checks a full user flow.
- Lower levels are faster and more precise. Higher levels are more realistic and more fragile.
- The test pyramid recommends many unit tests, fewer integration tests and few E2E tests.
- Tests tied to the UI break more often than tests on the logic.
