# ReScriptTest

> A lightweight test framework for ReScript

## Installation

Run the following in your console:

```console
$ yarn add --dev rescript-test
```

Then add `rescript-test` to your `bsconfig.json`'s `bs-dev-dependencies`:

```diff
 {
   "bs-dev-dependencies": [
+    "rescript-test",
   ]
 }
```

## Usage

```console
$ # All tests
$ retest test/**/*.bs.js
$ # Single file
$ retest test/MyTest.bs.js
```

## Basics

```rescript
open Test

let intEqual = (~message=?, a: int, b: int) =>
  assertion(~message?, ~operator="intEqual", (a, b) => a === b, a, b)

let stringEqual = (~message=?, a: string, b: string) =>
  assertion(~message?, ~operator="stringEqual", (a, b) => a == b, a, b)

test("Equals", () => {
  let a = 1
  let b = 1
  intEqual(a, b)
})

let isCharCode = (a, b) => {
  a->String.charCodeAt(0) === b
}

test("Custom comparator", () => {
  let a = "a"

  assertion(~message="Char code should match", ~operator="isCharCode", isCharCode, a, 97.0)
  assertion(~message="Char code should match", ~operator="isCharCode", isCharCode, a, 98.0)
})

type user = {username: string, id: string}

let userEq = (a, b) => a.id === b.id
let userEqual = (~message=?, a: user, b: user) =>
  assertion(~message?, ~operator="userEqual", userEq, a, b)

test("DeepEquals", () => {
  let a = {username: "user", id: "a"}
  let b = {username: "user", id: "a"}
  stringEqual(a.username, b.username)
  userEqual(a, b)
})
```

Outputs:

```sh
1/4: Equals
  PASS - No message
2/4: Custom comparator
  PASS - Char code should match
  FAIL - Char code should match
    ---
    operator: isCharCode
    left:  a
    right: 98
    ...
3/4: DeepEquals
  PASS - No message
  PASS - No message
4/4: Async
  PASS - No message

# Ran 4 tests (6 assertions)
# 3 passed
# 1 failed
```

## API

### Tests

- `test(name, body)`
- `testWith(~setup: unit => 'a, ~teardown: 'a => unit=?, name, body: 'a => unit)`
- `testAsync(name, body)`
- `testAsyncWith(~setup: unit => 'a, ~teardown: 'a => unit=?, name, body: ('a, cb) => unit)`

### Operators

- `throws(func, ~message: string=?, ~test: exn => bool=?)`
- `doesNotThrow(func, ~message: string=?)`
- `assertion(comparator, a, b, ~operator: string=?, ~message: string=?)`
- `pass()`
- `fail()`
- `todo(string)`

### Creating assertion shorthands

```rescript
let stringMapEqual = (~message=?, a, b) =>
  assertion(
    ~message?,
    ~operator="stringMapEqual",
    (a, b) => Belt.Map.String.eq(a, b, (a, b) => a === b),
    a,
    b,
  )
```

<details>
  <summary>Generic equal/deepEqual (not recommended)</summary>

Those that be useful to transition from an existing testing library, but we do not recommend polymorphic checks.

```reason
let equal = (~message=?, a, b) => assertion(~message?, ~operator="equal", (a, b) => a === b, a, b)

let deepEqual = (~message=?, a, b) =>
assertion(~message?, ~operator="deepEqual", (a, b) => a == b, a, b)
```

</details>

### Create tests with setup and teardown

The `setup` function returns a clean context for the test, the `teardown` function resets it.

```rescript
let testWithRef = testWith(~setup=() => ref(0), ~teardown=someRef => someRef := 0)

testWithRef("Setup & teardown", someRef => {
  incr(someRef)
  equal(someRef.contents, 1)
})
```
