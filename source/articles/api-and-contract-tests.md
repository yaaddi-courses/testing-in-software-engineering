# Testing APIs and contracts

**Covers:** API test, API, HTTP, contract test

## Testing at the seams between programs {#overview}
Think about how an app on your phone gets its data. It calls a server, and that server may call other services. Every one of these conversations follows rules, and many bugs come from one side changing the rules without telling the other. You can test the conversation directly, which is faster and more reliable than clicking through a screen. And a special kind of test protects the agreement between two teams. Let's see how.

## API {#api}
An API, or application programming interface, is a defined way for one program to ask another for something. A weather service may offer an API where you send a city name and get a forecast back. The API is the agreement: which requests are allowed, which answers come back and in what form.

## HTTP {#http}
Most APIs speak HTTP, short for HyperText Transfer Protocol, the language that browsers and servers use to exchange pages and data. A request names what you want (a path such as `/users/1`) and how (a method such as GET to read or POST to create). The response carries a status code and usually a body.

A few status codes appear constantly: 200 means success, 201 means something was created, 400 means the request was wrong, 401 or 403 mean you are not allowed, 404 means not found and 500 means the server failed. Tests often check the status code first, since it shows the outcome at a glance.

## API test {#api-test}
An API test sends a real request directly to a service and checks the response: the status code and the body. It needs no browser and no screen, so it is much faster and more stable than a test that drives a real browser. Even so, it checks a whole slice of the system, including routing, validation and the database.

Typical checks: a valid request returns 200 and the right fields, a missing field returns 400 with a helpful message, an unknown id returns 404 and a request without a login returns 401.

## contract test {#contract-test}
Teams change services independently, and that is where trouble starts. If the server renames a field from `name` to `fullName`, the apps that read `name` will break, and the server's own tests may all pass. A contract test checks that a service's API still matches what the programs that depend on it, its consumers, expect.

In consumer-driven contract testing, each consumer writes down what it relies on, such as "the response has an integer `id` and a string `name`". The service runs those expectations in its own build. A change that would break a consumer fails the service's build before it is released, and nobody has to run a full end-to-end test of every combination.

## A worked example {#example}
Let's use a pretend endpoint that returns a status code and a body. We test the behavior and the contract.

```python
def get_user(user_id):
    """A stand-in for GET /users/{id}: returns (status code, body)."""
    users = {1: {"id": 1, "name": "Ana", "email": "ana@example.com"}}
    if user_id in users:
        return 200, users[user_id]
    return 404, {"error": "not found"}


status, body = get_user(1)
assert status == 200 and body["name"] == "Ana"

status, body = get_user(99)
assert status == 404

# The contract: fields and types the consumers rely on
REQUIRED_FIELDS = {"id": int, "name": str, "email": str}
status, body = get_user(1)
for field, kind in REQUIRED_FIELDS.items():
    assert isinstance(body[field], kind), field
print("contract holds")
# contract holds
```

Here is how to read it:

1. The first two checks are API tests: a known user gives 200 and the right name, and an unknown user gives 404.
2. The contract lists only what the consumers rely on, and nothing more. That keeps it flexible, because the service may add fields freely.
3. If a developer renames `email` to `mail`, the loop raises a `KeyError` for `email` and the build fails, before any app is affected.

Now compare the costs. Finding the same problem with an end-to-end test would need a browser, a running app and a test account, and would take about 20 seconds. The contract check takes a few milliseconds.

## A common mistake {#mistake}
A common slip is checking the exact full body in every test, so that adding a harmless new field breaks many tests. Check the fields that matter.

A second slip is skipping contract tests because "we talk to that team every day". People forget, and a pipeline does not.

## Key points {#key-points}
- An API is the agreement for how programs ask each other for things, usually over HTTP.
- An API test sends a real request and checks the status code and the body, without a browser.
- A contract test checks that a service still gives its consumers what they expect.
- Check what matters and allow harmless additions.
