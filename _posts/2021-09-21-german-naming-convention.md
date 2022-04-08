---
layout: post
title: German / Dutch Naming Convention
date: 2021-09-21
---

There’s one thing that could make our life as software engineers much easier: better naming convention.

## Expect the Violent Psychopath

You’re trying to tell a story with your code. Your code should tell that story clearly, not cryptically, for an audience besides yourself. A good yardstick for deciding what kind of audience you are writing for is to imagine someone who has a familiarity with your domain but not your program’s take on the domain. I think programmers forget that, as they are authors, they have readers.

A famous and regularly quoted piece of advice, from the mailing list comp.lang.c in 1991, John F. Woods wrote:

> Always code as if the guy who ends up maintaining your code will be a violent psychopath who knows where you live. Code for readability.

It’s hard to put it better than that.

## Naming Tropes

There are some common naming conventions which are departures from plain English, usually in the interest of brevity:

- Abbreviations: when words are abbreviated such as `fct` for “function”, `dfn` for “definition”, `ctx` for “context.”
- It’s All Greek To Me: using simply `a`, `x`, etc. as in mathematics.
- “Hungarian” notation: any prefix or suffix notation in which a single letter is used to refer to a type or property of the variable, as in sigils like `$foo` (“scalar foo”), `lpszFoo` (“long pointer string zero-terminated”), or `fooL` (list of foo).
- Acronyms: using initial letters to refer to concepts: `throwVE` (“throw validation error”).

Most of these are unnecessary and/or harmful.

## It’s All Greek To Me

A word on this convention. Single letter naming comes from mathematical tradition; it means “there isn’t a good noun for this because it’s general”. A person of X height. In some cases, this is actually reasonable. Consider:

`identity x = x`

The identity function isn’t enhanced by calling its parameter thing; it literally doesn’t matter what it is, especially in some typed languages. In fact, one could argue that it’s harmful to try to using a meaningful English name.

However, anywhere that your variables have some meaning, by using “Greek convention”, you’re throwing away information that could help someone to digest your code better. You’re not trying to fit your code on a napkin.

## German / Dutch Naming Convention

This is what I consider good naming convention. I discovered this convention while working with a German colleague, who, I’d always joked, uses long variable names, and almost never abbreviates anything. However, the more I read his code, the more I realised I was able to read the story he was trying to tell, and appreciated it a lot: Using as many words as necessary to clearly name something. Everything.

I called this “German” naming convention although same applies for Dutch, as a reference to the fact that the German language is known for its compound words, which can become comically long and specific at times. Some examples include, Betäubungsmittelverschreibungsverordnung (“regulation requiring a prescription for an anaesthetic”), Rechtsschutzversicherungsgesellschaften (“legal protection insurance companies”), and the 1999 German “Word of the Year”: Rindfleischetikettierungsüberwachungsaufgabenübertragungsgesetz (“beef labelling regulation and delegation of supervision law”).

Don’t write `fopen` when you can write `openFile`. Write `throwValidationError` and not `throwVE`. Call that name `function` and not `fct`. That’s German naming convention. Do this and your readers will appreciate it.

## Isomorphic Naming

This convention complements German naming convention completely.

Isomorphic naming is to say that the name of the variable is the same form of the name of the type. A simple heuristic, in other words: just use the name of the type.

Here’s a real sample where better naming convention would make this easier to read without being a cryptographer:

```haskell
updateColExp
  :: QualifiedTable -> RenameField -> ColExp -> IO ColExp
updateColExp qt rf (ColExp fld val) =
  ColExp updatedFld <$> updatedVal
  ...
```

Look at this naming convention. This may be appropriate if you’re in some kind of code golfing competition, but I can’t even pronounce these names. Applying the type-based naming heuristic, we get:

```haskell
updateColumnExpression
  :: QualifiedTable -> RenameField -> ColumnExpression -> IO ColumnExpression
updateColumnExpression qualifiedTable renameField (ColumnExpression field value) =
  ColumnExpression updatedField <$> updatedValue
  ...
```

Look, it’s readable, plain English! Isn’t this a huge improvement? Any maintainer reading this code can read each variable and know what it is. I can even pronounce the names out loud.

Note that this convention only works well when your types are well-named too, by German naming convention.

Original post can be found at [Chris Done's site](https://chrisdone.com/posts/german-naming-convention/).