# Statements as expressions

## Status

Champion(s): _Currently none_

Author(s): Scotty Jamison

Stage: -1

## Motivation

The purpose of this proposal is to provide intuitive constructs that enable the use of Javascript in an expression-oriented format. It is intended to accomplish the same objectives as the [do expression proposal](https://github.com/tc39/proposal-do-expressions) but in a much lighter and straight-forward fashion.

## Description

Do expressions take the route of trying to allow as many statements as possible in an expression position, then black-listing scenarios that don't make sense. This proposal instead strives to find the specific statements that people actually need to use as an expression, and creating new constructs to provide them. Turns out, there's not very many new constructs needed to complete the objectives that do expressions attempt to solve.

### Expression-if

One of the most common motivating examples of do expressions is it's ability to do an if statement in an expression position. This proposal will seek to simply add a new `with if` construct, that provides this same functionality.

Here are some examples that have been pulled from the do-expression proposal and retrofitted to use this proposal's syntax instead:

```javascript
let x =
  with if (foo()) f()
  else if (bar()) g()
  else h()
```

```javascript
return (
  <nav>
    <Home />
    {
      with if (loggedIn) (
        <LogoutButton />
      ) else (
        <LoginButton />
      )
    }
  </nav>
)
```

The syntax for "with if" is as follows:

```
with if (<condition>) <expression> [else if (<condition>) <expression>]* else <expression>
```

Note that the final else clause is required.

### Expression-try

One of the biggest pain-points of try-catch is the fact that you have to use an outside, mutable declaration to store any useful value generated from within the try catch. Compare these two code snippets, one using try-catch as it stands in Javascript today, and the other using the new expression version of try-catch, "with try":

```javascript
// Before
let user
try {
  user = await getUser()
} catch (err) {
  if (err instanceof NotFound) {
    user = null
  } else {
    throw err
  }
}

// After
const user = with try (
  await getUser()
) catch (err) (
  err instanceof NotFound
    ? null
    : throw err // This uses a throw expression, from the throw expression proposal.
)
```

The syntax for "with try" is as follows:

```
with try <expression> catch (<identifier>) <expression>
```

Note that a finally block is not allowed.

### Expression-declarations

The final and most important piece to the puzzle is the ability to provide declarations in an expression position. This will be provided through a "with declaration" syntax as follows:

```javascript
const result =
  with username = getUsername(params)
  with id = await getUserId(username)
  do getGroups(id)

// ... is the same as ...

const result = await (async function() {
  const username = getUsername(params)
  const id = await getUserId(username)
  return getGroups(id)
})
```

Any declaration created in a "with" will only be available within the rest of the current with/do expression. Anything after the "do" becomes the completion value of the with/do expression.

The syntax for a with declaration is as follows:
```
[with <identifier> = <expression>]+ do <expression>
```

i.e. you must have one or more with bindings, followed by "do", then an expression.

## Comparison

Here are some ways in which rescript/reasonML solves these same issues:
```rescript
// if as expression
let result = if true {
  sideEffect()
  b // result will be set to b
} else {
  c
}

// try/catch as expression
let result =
  try {
    getItem([1, 2, 3])
  } catch {
  | Not_found => 0 // Default value if getItem throws
  }

// declarations in expression position
let result = {
  let x = 23
  let y = 34
  x + y // result will be assigned to x + y
}
```

You'll note that the proposed "with if" syntax is very similar to rescript's if syntax. It's interesting to note that they provide syntax for both if/else and ternaries. Their try/catch also looks similar to this current proposal, except rescript allows you to catch specific error types. Their block-scope behavior aligns closer with how do-expressions function, and is the type of thing we're hoping to stay away from.

Examples from Elm:
```elm
// if as expression
if key == 40 then
  n + 1
else if key == 38 then
  n - 1
else
  n

// Elm has no concept of try/catch/throw.

// declarations in expression position
let
  twentyFour = 3 * 8
  sixteen = 4 ^ 2
in
  twentyFour + sixteen
```

The declaration syntax in this proposal aligns closer to that of Elm's, and many other functional languages. First you provide your different declarations, then you provide an expression in which you use those declarations.


## Q&A

**Q**: What about using throw in an expression position? Or switch in an expression position?

**A**: Both of those are already being handled in separate repos. See [throw-expression](https://github.com/tc39/proposal-throw-expressions) and [pattern matching](https://github.com/tc39/proposal-pattern-matching).

**Q**: What about using break, continue, return, loops, etc in an expression position?

**A**: The reasons for not including these have all been stated in the original TC39 form post [here](https://es.discourse.group/t/statements-as-expressions/894). Feel free to take a peek to see some of the rational behind the current proposal, and feel free to open a new issue if you think any of these would be useful as an expression.

**Q**: I don't like the syntax/semantics of this proposal, what if we did...

**A**: Go ahead and share your suggestion in a new issue! The overall idea of this proposal is very much in flux, and there's many different directions we can take. The solution presented here is just one possible route we could take.