# Test doubles: stubs, mocks, fakes and spies

**Covers:** stub, mock, fake, spy, test double, mocking

## Standing in for the real thing {#overview}
Real code rarely works alone. It calls a payment service, reads from a database, sends an email or asks for the current time. In a test, those dependencies are a problem: they can be slow, cost money, fail at random or be hard to set into the state you need. The fix is to replace them with stand-ins for the length of the test. There are several kinds, and choosing the right one makes tests clearer and more reliable.

## test double {#test-double}
Any object that stands in for a real dependency during a test is a test double. The name comes from the stunt double in films, who takes the place of the actor for dangerous scenes. Stubs, mocks, fakes and spies are all test doubles, and each is chosen for a different purpose.

## stub {#stub}
A stub is the plainest stand-in. It returns a fixed, prepared answer whenever it is called, and it does not check how or whether it was used. Use it when the code under test needs some input from a dependency and you want to control that input: "the shipping service always quotes 5".

## mock {#mock}
A mock does what a stub does, and in addition it checks that it was called in the right way: with the expected arguments, the right number of times, in the right order. The check is part of the test's result. Use it when the thing you care about is the interaction itself, such as "an email was sent to this address".

## fake {#fake}
A fake is a working but simplified version of a real dependency. A common example is an in-memory database instead of a real one. It behaves like the real thing in the ways your code needs, but it is quick and needs no set-up. Use it when the code makes many calls and you want realistic behavior without the real system's cost.

## spy {#spy}
A spy records the calls made to it, so the test can look at them afterward. It usually wraps the real thing or a plain stand-in and lets the call go through, then reports what happened. The difference from a mock is when you decide what to check: a mock states its expectations first, and a spy lets you inspect the history after the fact.

## mocking {#mocking}
Swapping a real dependency for a test double is called mocking. You usually do it to isolate the code under test from something slow, unreliable or hard to control. It is a useful tool, but overuse hurts. A test with too many mocks may check only that the code calls the mocks in a certain way, and not that the real system works. Such a test can pass while the real system is broken.

Some good habits: mock at the edges of your system, such as the network, the clock and the file system, and not your own internal code. Prefer fakes for things you call often. And keep at least some integration tests that use the real dependency.

## A worked example {#example}
A function adds shipping to a cart total. Let's test it three ways.

```python
from unittest.mock import Mock


def total_with_shipping(cart, shipping_client):
    cost = shipping_client.quote(len(cart))
    return sum(cart) + cost


# Stub: a fixed answer
stub = Mock()
stub.quote.return_value = 5
print(total_with_shipping([10, 20], stub))
# 35

# Mock or spy behavior: check how it was called
stub.quote.assert_called_once_with(2)
print(stub.quote.call_args)
# call(2)


class FakeShipping:
    """A working, simplified version: a flat rate per item."""

    def quote(self, items):
        return items * 2


print(total_with_shipping([10, 20], FakeShipping()))
# 34
```

Let's read it:

1. As a stub, the shipping service always answers 5, so the total is 10 + 20 + 5 = 35.
2. The same object then acts as a mock: `assert_called_once_with(2)` fails the test if the function did not ask for a quote for exactly two items. The recorded call is also a spy's output, which you can print or examine.
3. The fake has real logic: 2 items at 2 each is 4, so the total is 10 + 20 + 4 = 34.

The stub tests the arithmetic, the mock tests the conversation with the service and the fake lets you test many carts without an external service.

## A common mistake {#mistake}
A common slip is mocking so much that the test copies the implementation line by line. Then every refactor breaks the test even though the behavior is correct. Test what the code does, not how it does it, and replace only what you must.

## Key points {#key-points}
- A test double is any stand-in for a real dependency.
- A stub returns canned answers, a mock also checks the calls, a fake is a simple working version and a spy records calls.
- Mock at the edges of the system, and not your own internals.
- Over-mocking can pass tests while the real system is broken.
