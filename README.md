# Idrall

[Dhall](https://dhall-lang.org) bindings for [Idris](https://www.idris-lang.org).

Parse, evaluate, check/infer types of Dhall expressions.

## Status

Very much a work in progress, with many thing missing. The plan is to make an end to end compiler for a small goofy subset of Dhall, and gradually and features.

## Features

Features marked with a tick should work for parsing, type checking and evaluation.

- [x] Fuctions
- [x] Types
  - [x] Bool
  - [x] Natural
  - [x] Integer
  - [x] List
  - [x] Optional
  - [x] Equivalent
  - [x] String
  - [x] Records
  - [x] Union
  - [x] Double
- Operators
  - [x] `&&`
  - [x] `#`
  - [x] `.` (Field operator)
  - [ ] `\\`
  - [ ] `/\`
  - [ ] `::`
  - etc.
- Builtins
  - [x] `Natural/isZero`
  - [x] `List/head`
  - [x] `Integer/negate`
  - [ ] Everything else
- Imports
  - [x] local files
  - [ ] Env
  - [ ] http
- [ ] `x@1` style variables
- [ ] Anything to do with caching
- [ ] CBOR representation
- [ ] The rest of this list

## Dependencies

[idris2](https://github.com/idris-lang/Idris2)

Not required, but some of the Makefile commands use [`rlwrap`](https://github.com/hanslub42/rlwrap) to make the Idris2 repl behave better.

## Installation

```
make install
```

## Tests

```
make test
```

## Implementation details

### Type checking

Type checking and inference (aka synthesis) in Dhall is covered by [these rules](https://github.com/dhall-lang/dhall-lang/blob/master/standard/type-inference.md). The rules are implemented here using a technique called Normalisation by Evaluation (NbE). It is described in [this paper by David Christiansen](http://davidchristiansen.dk/tutorials/implementing-types-hs.pdf), and was also used by [@AndrasKovaks](https://github.com/AndrasKovacs) in [their branch](https://github.com/dhall-lang/dhall-haskell/commits/nbe-elaboration) on `dhall-Haskell` (I found [this commit](https://github.com/dhall-lang/dhall-haskell/commit/627a6cdea0170336ff08de34851d8bdf5180571d) particularly useful).

The general idea of NbE is that you have a data structure that represents the raw syntax Language, which is called `Expr` here. Expressions can be literals (`3`, `True`), types (`Natural`, `Bool`, `Type`), functions, builtins, operators, etc. You evaluate the `Expr` to a data structure that only represents expressions that cannot be reduced further, called `Value` here. Eg, the expression `True && False` can be represented in an `Expr`, but as a `Value` would be reduced to `False`. 

To convert a `Value` back to an `Expr` is called "reading back" or "quoting".

To normalise an `Expr` you evaluate it to a `Value`, then read it back to a `Expr`, this which ensures it's fully normalised.

Type synthesis (or type inference) takes an `Expr` and returns its type as a `Value`. 

Type checking checks an `Expr` against a type given as a `Value`. A `Value` is synthesised for the type of the `Expr`, and both this `Value` and the provided type `Value` are read back to `Expr`s. This ensures there are no reducible expressions in either. Now you can compare the types using an alpha equivalence check to see if they match.

That was a very brief, potentially wrong introduction to the NbE technique used. I'm glossing over a bunch of details about closures, neutral values, the environment/context, etc. but this should be enough to get started with. See the above paper for a full description, and check out the code to see it in action.

## Contributions

Any contributions would be appreciated, and anything from the missing list above would be a good place to start.

### Examples of adding language features

Adding features generally means editing the `Expr` and `Value` types, the parser, the `eval`/`check`/`synth` functions, the tests etc.

As an example, the `List` type was added via [#1](https://github.com/alexhumphreys/idrall/pull/1), and literal values of type list (`[1, 2, 3]`) were added via [#2](https://github.com/alexhumphreys/idrall/pull/2). For an example of an operator, [#3](https://github.com/alexhumphreys/idrall/pull/3) adds the `#` operator for appending lists.

## Idris1 compatibility

There is an [`idris1` tag](https://github.com/alexhumphreys/idrall/releases/tag/idris1) which is the last confirmed commit that works with idris1. It's got all the dhall types and not much else, so if you're desperate for a Dhall implementation for idris1 it may help, but realistically you're gonna need the Idris2 version.

## Future work

- Add the things from the missing list above
- Use dependent types to prove field names in values are elements of their Unions/Records
- Improved parsing (Not really sure what I'm doing here)
- Think about what api/types to expose so as to make this as nice as possible to use
- Scope checking as found in [Tiny Idris](https://github.com/edwinb/SPLV20)
