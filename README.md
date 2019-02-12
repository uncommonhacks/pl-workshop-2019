# WAT: An introduction to PLT
- https://www.destroyallsoftware.com/talks/wat
- PLT, or Programming Language Theory, is the study of everything wrong with
  what you just saw.
  - This workshop will cover the fundamentals of PLT
  - By the end, you will be equipped to build your own language!

# A concrete example
- Consider the program `1 + 2 + 3`.'
- How do we take this from a string input to meaningful output?
  - There are distinct phases of compilation.

## Scanning
- The scanner takes the input and transforms it into representable data.
  - Specifically, it transforms a string into a **token list**.
- For example, the result of the scanner on our program would be:
  - [1, Plus, 2, Plus, 3]
- The result of scanning is fed into the parser.

## Parsing
- The parser takes the scanner's output and structures it.
  - Specifically, it transforms a token list into an **abstract syntax
    tree**, or AST.
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
  - For example, in our program from above:
```
 Plus
 /  \
1  Plus
   /  \
  2    3
```
    - The typechecker begins by seeing `Plus` at the root of the AST. So, it
      knows that both sides must have type `Int` for the expression to make any
      sense. To do this, we recursively typecheck both sides of the expression.
  - How about in the case of a type error?
```
 Plus
 /  \
1  Plus
   /  \
  2   "hi"
```
    - Again, the typechecker begins by deducing that both sides of the
      expression must have type `Int`. But, when recursively checking the
      right-hand side, we see that indeed one of the operands to `Plus` isn't an
      `Int`, which causes a type error.

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


# The phases of compilation
- We'll be writing an interpreter. What does that entail?
- You need to read a source program, and manipulate it to produce output.
- Phases: scan, parse, typecheck, evaluate
    - Goals:
    - Scan: read in source code, character by character, and produce something
      that can be operated on programmatically - a token list
    - Parse: take that token list, and produce a syntax tree that formally
      describes the program
    - Typecheck: read in a syntax tree and make sure that it doesn't contain
      errors
    - Evaluate: traverse a syntax tree and produce an output

# Scanning
- Scanning is the "dumb" phase. Not much interesting happens here - the idea is
  to "normalize" the input into something that's nice to operate on.
- But what do we even scan? PLs begin with **concrete syntax**.
- Concrete syntax is defined using Backus-Naur Form, or BNF.
    - Step through BNF for lispy boolean algebra: `True`, `False`, `&&`, `||`,
      `!`
    - Hit on metavariables
- Scanning simply takes something written in concrete syntax and produces
  something we can operate on programmatically
    - SML makes this easy: define `datatype token`

# Parsing
- Here's where interesting things start to happen!
- So let's suppose we have this program: `(&& (! False) False)`
    - How do we turn this into something we can operate on?
- Goal: produce an **abstract syntax tree**, or AST
    - Draw out the AST of this program
    - To understand the difference, note that the parens don't appear in the
      abstract syntax
    - SML also makes this easy: define `datatype term`
- There will be lots of recursion here, as you'll need to find a way to "gobble
  up" `! False` as the first parameter to the `&&`.

# Evaluation
- Goal: take in an abstract syntax tree and produce its output.
- Consider our example program: writing a fully general function here to
  evaluate recursive structures seems... unwieldy.
- So, define a **small-step evaluation relation**.
    - Write out the judgment form rules for `!`, `&&`
        - Point out short-circuiting in `&&`
- Once you have a small-step evaluation function, the evaluation function itself
  is pretty much trivial. You just step until you can step no more.
    - Once you can step no more, you've reached a **normal form**. Certain
      normal forms are said to be **values**: those are normal forms that the
      language designer has determined are reasonable program outputs.
    - Non-value normal forms are called **stuck terms**. Stuck terms represent
      errors! For example, a segfault in C is a stuck term (memory access that
      can't be executed).

# Typechecking

- Goal: eliminate stuck terms
- We need to first take a detour and talk about typechecking in a vacuum.
- Goal: assign types to programs.
    - Typechecking rules are defined in judgment form, similar to small-step
      evaluation rules.
    - Write out the judgment form rules for typechecking `True`, `False`, `!`,
      and `&&`.
- **Type soundness** is the sickest fucking thing on planet earth.
    - A type soundness theorem is a theorem stated for a specific programming
      language, evaluation relation, and typechecking rules. A type soundness
      theorem contains two lemmas:
        - Progress: if a term can be assigned a type, and that term is not a
          value, then it can take a step somewhere.
        - Preservation: if a term can be assigned a type and there exists a term
          that it can take a step to, then that new term has the same type.
    - If proven successfully, it states the following:
        - Any term which can be assigned a type will eventually step to a value.
    - This is **wicked fucking lit**. In a formal setting, i.e. a language that
      has a proven type soundness theorem, **any program that passes the
      typechecker is free of runtime errors.**

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
