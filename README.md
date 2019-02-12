# WAT: An introduction to PLT
- https://www.destroyallsoftware.com/talks/wat
- PLT, or Programming Language Theory, is the study of everything wrong with
  what you just saw.
  - This workshop will cover the fundamentals of PLT
  - By the end, you will be equipped to build your own language!

# A concrete example
- Consider the program `1 + 2 + 3`.
- How do we take this from a string input to meaningful output?
  - There are distinct phases of compilation.

## Scanning
- The scanner takes the input and transforms it into representable data.
  - Specifically, it transforms a string into a **token list**.
- For example, the result of the scanner on our program would be:
  - `[1, Plus, 2, Plus, 3]`
- The result of scanning is fed into the parser.

## Parsing
- The parser takes the scanner's output and structures it.
  - Specifically, it transforms a token list into an **abstract syntax
    tree**, or **AST**.
- For example, the result of the parser on the scanner's output from above would
  be:

```
 Plus
 /  \
1  Plus
   /  \
  2    3
```

## Typechecking
- The typechecker takes the parser's output and makes sure that all the types
  match up.
  - Specifically, it transforms an AST into a **type attempt**.
- This is probably the hardest output type to understand, so here are examples.
  - `1 + 2 + 3` results in `Int`
  - `1 + 2 + "hi"` results in a type error
- How does it accomplish this?
  - By recursively verifying the types of subexpressions.
    - For example, in `1 + 2 + 3`: the typechecker begins by seeing
      `Plus` at the root of the AST. So, it knows that both sides must have type
      `Int` for the expression to make any sense. To do this, we recursively
      typecheck both sides of the expression.
  - How about in the case of a type error?
    - In `1 + 2 + "hi"`, again, the typechecker begins by deducing that both
      sides of the expression must have type `Int`. But, when recursively
      checking the right-hand side, we see that indeed one of the operands to
      `Plus` isn't an `Int`, which causes a type error.

## Evaluation
- The evaluator takes the parser's output and, well, evaluates it.
  - Specifically, and perhaps surprisingly, it transforms an AST into **another
    AST**.
- How does it do this?
  - By taking steps. When it's not possible to take a step, you're done.
- For example, the steps that the evaluator would take on the AST from above
  would be:
```
 Plus
 /  \
1  Plus
   /  \
  2    3
```
```
 Plus
 /  \
1    5
```
```
6
```


# Time to formalize!
## Scanning
- PLs begin with **concrete syntax**, which is defined using Backus-Naur Form,
  or BNF. Here's an example:
```
t ::=
  | 0, 1, ...
  | t + t
  | t * t
  | true
  | false
  | !t
  | t && t
  | t || t
  | (t)
```
- Earlier we mentioned that scanning takes a string and produces a token list.
  So what's a token?
  - A token is simply a programmatic representation of the components of a BNF
    term. For example, in Standard ML, we could write the following:
```
datatype token =
    Numeric of int
  | Plus
  | Times
  | True
  | False
  | Bang
  | AndAnd
  | OrOr
  | LParen
  | RParen
```
- Now, the scanner will *try* to transform its input into a token list, or fail
  if you give it input that's not part of the language's concrete syntax. For
  example, `1 + qoi3egwbs` would be a scan error.

## Parsing
- Once we have a token list, we need to produce an AST. Just like with tokens,
  we need a way to represent ASTs.
  - In Standard ML, we could write the following:
```
datatype term =
    Numeric of int
  | Plus of term * term
  | Times of term * term
  | True
  | False
  | Not of term
  | And of term * term
  | Or of term * term
```
- Now, the parser will *try* to transform its input into an AST, or fail if you
  give it valid tokens that don't make sense. For example, `1 (5)` would be a
  parse error, even though it would scan correctly.

## Evaluation
- Once you have an AST, you need a way to evaluate on it. But evaluating
  recursive structures is complex; so we reduce the problem to *taking steps* on
  recursive structures.
  - A small-step evaluation relation is defined by writing rules in *judgment
    form*. In fact, everything we've covered so far can be written in judgment
    form, but it's less intuitive for the earlier phases.
- Here are some judgment form rules:
```
--------------
!false -> true


--------------
!true -> false
```
- These two rules mean that without any predicate, `!false` steps to `true` and
  `!true` steps to `false`.
  - But this isn't sufficient! If we could only step on the negation of boolean
    literals, we'd be very limited. We want to be able to do something like
    `!(!true)` and have it evaluate to `false`.
  - To do this, we need a more complex rule:
```
 t -> t'
---------
!t -> !t'
```
- This rule means that if it is possible for `t` to step to `t'`, then `!t` can
  step to `!t'`.
  - Suppose we're looking at `!(!true)`. Then `t` is `!true`. Since `t` can step
    to `false` (i.e. `t'` is `false`), then `!(!true)` can step to `!false`.

- Once you have a small-step evaluation function, the evaluation function itself
  is pretty much trivial. You just step until you can step no more.
    - Once you can step no more, you've reached a **normal form**. Certain
      normal forms are said to be **values**: those are normal forms that the
      language designer has determined are reasonable program outputs.
    - Non-value normal forms are called **stuck terms**. Stuck terms represent
      errors! For example, a segfault in C is a stuck term (memory access that
      can't be executed).

## Typechecking
- Goal: eliminate stuck terms
  - We need to first take a detour and talk about typechecking in a vacuum.
- Goal: assign types to programs.
    - Typechecking rules are defined in judgment form, similar to small-step
      evaluation rules. Here are some examples:
```
t: Bool
--------
!t: Bool
```
- This means that if `t` has type `Bool`, then `!t` also has type `Bool`.

```
t1: Int, t2: Int
----------------
  t1 + t2: Int
```
- This means that if both `t1` and `t2` have type `Int`, then `t1 + t2` also has
  type `Int`.

- What do we get from all of this? Why bother?
  - **Type soundness** is the sickest fucking thing on planet earth.
    - A type soundness theorem is a theorem stated for a specific programming
      language, evaluation relation, and set of typechecking rules. A type
      soundness proof contains two lemmas:
        - Progress: if a term can be assigned a type, and that term is not a
          value, then it can take a step somewhere.
        - Preservation: if a term can be assigned a type and there exists a term
          that it can take a step to, then that new term has the same type.
    - If proven successfully, it states the following:
        - Any term which can be assigned a type will eventually step to a value.
    - This is **wicked fucking lit**. In a formal setting, i.e. a language that
      has a proven type soundness theorem, **any program that passes the
      typechecker is free of runtime errors.**
      - Now, there's a small asterisk here; we can only prevent a certain class
        of runtime errors. For example, if you have a program where you add 5
        when you meant to multiply 5, there's no way the typechecker can catch
        that.
      - But on the flipside, this is a *huge* category of errors that we can
        catch! For example, segfaults would be impossible if C were type-sound.

- - - -

# Your language here!

- This language is super, super tiny. It has one type, and doesn't even have
  conditional branching.
- I recommend your language include at minimum:
    - Boolean constants and unary/binary operators (so, the example language
      we've been using)
    - A conditional (an if statement)
    - A system of numbers! Peano arithmetic (i.e. `Zero`, `Succ`, and `Pred`) is
      a good choice. This makes determining whether or not something is a value
      nontrivial!
- If you end up being able to implement all this - and to be clear, I will be
  shocked if anyone can - I recommend implementing the following:
    - Tuples
    - Option types
    - Lists
