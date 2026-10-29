# Security and accessibility testing

**Covers:** security testing, SQL injection, SQL, accessibility testing

## Two kinds of failure that users cannot report {#overview}
Most bugs are found because someone notices that something is wrong. Two important kinds go unnoticed by almost everyone: weaknesses that an attacker can exploit, and barriers that block people who use the product differently. Ordinary use will never reveal either, so both need tests on purpose.

## security testing {#security-testing}
Security testing looks for weaknesses before an attacker does. It asks questions such as: can someone read another user's data by changing a number in the address, does a page that should need a login really require one, are passwords stored safely, and does the program handle hostile input safely?

Part of it can be automated. Scanners look for known vulnerable libraries and common mistakes, and tests can check that protected endpoints return an error without a valid login. Part of it needs a person who thinks like an attacker. It works best when done regularly, and early, since fixing a weakness in design is cheaper than after a breach.

## SQL {#sql}
To understand one famous attack, you first need SQL, or Structured Query Language, the standard language for asking questions of a relational database, such as "give me the secret for the user called Ana". Most applications build such queries from what a user types, such as a name in a search box. That is where the risk begins.

## SQL injection {#sql-injection}
Suppose someone types malicious code into an input, and the program runs it as part of a database query. That attack is SQL injection. If a program builds the query by pasting the user's text into a string, the attacker can change the meaning of the query and read or alter data they should not see.

The fix is well known: never build queries by joining strings. Use parameters, also called placeholders, so that the database always treats the input as data, never as code. A security test sends crafted inputs such as `' OR '1'='1` and checks that nothing unexpected happens.

## accessibility testing {#accessibility-testing}
Accessibility testing checks that the software can be used by everyone, including people who are blind, deaf, color-blind, who cannot use a mouse or who have motor or cognitive differences. Typical checks: can a screen reader announce every control, can the whole app be used from the keyboard, do images have text descriptions, and is there enough contrast between text and background?

The Web Content Accessibility Guidelines set measurable targets. For normal-sized text, the contrast ratio should be at least 4.5 to 1. Automated tools catch about a third of the issues, and a person testing with a keyboard and a screen reader finds the rest.

## A worked example {#example}
First, SQL injection against a small in-memory database.

```python
import sqlite3

db = sqlite3.connect(":memory:")
db.execute("CREATE TABLE users (name TEXT, secret TEXT)")
db.execute("INSERT INTO users VALUES ('ana', 'a-secret'), ('ben', 'b-secret')")

name = "nobody' OR '1'='1"

unsafe = f"SELECT secret FROM users WHERE name = '{name}'"
print(len(db.execute(unsafe).fetchall()))
# 2

safe = "SELECT secret FROM users WHERE name = ?"
print(len(db.execute(safe, (name,)).fetchall()))
# 0
```

Let's read it:

1. The attacker's "name" is not a name. It ends the quoted text and adds `OR '1'='1'`, which is always true.
2. In the unsafe version, the query becomes "where name is 'nobody' or true", so it returns both secrets: 2 rows.
3. In the safe version, the whole string is treated as one name. No user has that name, so 0 rows come back.

A security test would assert that the result for this input is 0, and would fail for the unsafe code.

Second, an accessibility check on color contrast. Black text on white has a ratio of 21 to 1, the maximum. Medium gray (#777777) on white measures 4.48 to 1, a little under the 4.5 target for normal text, even though it looks fine to many people. A tool that computes the ratio catches this at once, while a quick look does not.

## A common mistake {#mistake}
A common slip is treating security and accessibility as a final-week activity. Fixing them late is expensive, and some problems are built into the design. Add a few automated checks to the build from the start.

A second slip is relying only on automated scanners, which miss many problems. Add a manual pass.

## Key points {#key-points}
- Security testing looks for weaknesses before attackers find them.
- SQL injection happens when user input becomes part of a query. Use parameters to stop it.
- Accessibility testing checks that people with different abilities can use the product.
- Automated checks find some problems. A manual pass finds the rest.
