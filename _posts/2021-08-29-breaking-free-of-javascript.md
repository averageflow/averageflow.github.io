---
layout: post
title: Breaking free of JavaScript
date: 2021-08-29
---

![Javascript has us prisoners for front-end development](https://res.cloudinary.com/dehs6irlh/image/upload/v1630256482/jjba-site/blog/breaking-free-from-js/1_hTGEAKTrOdt_vRzNUCcgRA_qgtbyc.png){: .center-image }


Javascript has a stranglehold on all Front End Development. If you write code for the browser, then it’s most likely written directly in Javascript or its very close cousin TypeScript. The problem is that Javascript is a terrible language.

TLDR: You should break free from Javascript by learning PureScript which compiles to Javascript.

Typescript and other attempts to curtail Javascript are about as effective as a band-aid on a puncture wound. One might argue that it’s better than nothing but eventually, you’re still going to bleed out.

The language has undergone many changes since its initial development, which consisted of a whopping 10 days, but all of these changes are just polishing a turd.

Javascript is a veritable Smorgasbord of languages paradigms. It has some Object Oriented, Functional and Procedural features all mixed together in an unpalatable Schizophrenic Goulash (mixed metaphor intended).

There are more bad parts of Javascript than good parts and anyone working in Javascript on a daily basis will attest to the fact that being a good Javascript Developer is more about knowing what NOT to do.

Seasoned Javascript Developers have a litany of language constructs that they routinely avoid like the plague lest they fall victim to the plethora of runtime exceptions that are routinely encountered in production, e.g. the dreaded “undefined is not a function”.

Javascript’s reign is supreme thanks to its monopolistic hold on the Browser. All previous attempts to extricate the Browser from the tyranny of Javascript have long since failed leaving most leery of attempting yet another failed coup d’état.

## Freedom or Death

![Freedom or Death](https://res.cloudinary.com/dehs6irlh/image/upload/v1630256482/jjba-site/blog/breaking-free-from-js/1_0xnuE8J8c0Ivwe_ud5i5xA_qk6ehg.jpg){: .center-image }

What options do we have?

We could decide not to develop applications for the Browser. I like to think that we should develop mostly on the server-side by default, unless the situation requires very advanced client-side features and state. I would personally recommend Go and Haskell as strong candidates for server side applications. This of course could be done in a number of different languages.

We could create an Open Source Browser that would allow for other languages or perhaps it could have a built-in language that’s “better” than Javascript.

But our Browser would be completely incompatible with every single web site on the planet. This may appear to be Freedom at first but it most definitely is Death. No one but the most fervent would adopt this.

We could develop a Browser Extension that allows for a better developer experience. This too has been tried before and since we’re reaching a near monopoly in Browser development by only the largest of companies, our initial taste of Freedom could, on a whim of a corporate giant, be transformed into sudden Death.

Most affected by an oppressive environment are too busy trying to survive in the current climate than to overturn it.

Revolutions are rare, dangerous and costly but subversion isn’t.


## If you can’t beat ’em, join ’em (sort of)

Javascript’s stranglehold on the Browser isn’t a new phenomenon. We’ve seen this very thing before. In fact, it’s rampant in the hardware world.

A Microprocessor has a single instruction set that can never be superseded by any other. Not without completely replacing the hardware.

Yet, we don’t call for revolution but instead work to insulate ourselves from the deficiencies of such a system. We did this over half a century ago when we created high-level languages.

Any flaws or complexities in the underlying architecture are squelched by an abstraction layer that frees us from having to regularly consider the pitfalls.

By writing a compiler, we freed ourselves from the tyranny of a single platform and by using this same approach, we can free ourselves from our current dilemma.


## A Horse of a Different Color

![A Horse of a Different Color](https://res.cloudinary.com/dehs6irlh/image/upload/v1630256482/jjba-site/blog/breaking-free-from-js/1_g4vok0cblQYGTiK5_k_Nmg_hyftb6.jpg){: .center-image }

What we want is a way to write Javascript without having to write Javascript. To do that, we’re going to need a Transpiler.

A Transpiler will compile code in one language and produce code in another. Technically, CoffeeScript, TypeScript and Babel are Transpilers but they start with nearly Javascript and produce Javascript.

These solutions do not give us the benefits that we’re hoping for. This is the equivalent of writing in Assembly Language instead of Machine Code.

What we want is a whole new language. One that avoids all of the terrible design decisions of Javascript. There are many languages that transpile to Javascript that are far superior to Javascript.

I’m going to concentrate on Functional Programming Languages only because it’s becoming very clear that Functional Programming is the future of our industry.

This is evident by the mass adoption of Functional Features in today’s most popular languages. This is historically what’s been seen right before a major paradigm shift is about to occur in the software industry.

For the curious, Richard Feldman does a wonderful job of making this argument in this entertaining and illuminating talk The Next Paradigm Shift in Programming.

### Go

GopherJS is an honourable mention and could be exactly what you search if you are into Go: [see GopherJS on GitHub](https://github.com/gopherjs/gopherjs)


### Elm

![Elm logo](https://res.cloudinary.com/dehs6irlh/image/upload/v1630256482/jjba-site/blog/breaking-free-from-js/1_yok_lg3cm73w-eeJttUY2A_xzgznt.png){: .center-image }

Elm is a great beginner language for Functional Programming. The ecosystem is mature and I have personally been responsible for a team that put over 160K lines of Elm code into production without a single line of Elm code producing a Runtime Error.

Elm is a dialect of ML and a very small subset of Haskell.

If you want to dip your toe into the Statically Typed, Purely Functional Programming world, then Elm can be a great starting point (https://elm-lang.org/).

Unfortunately, Elm’s lack of power quickly shows as your application becomes complex and the resources for learning are somewhat limited. The go-to book for learning Elm is Elm in Action.

### ReasonML

Facebook uses Functional Programming Languages, one of which is Haskell, the granddaddy of them all. The other notable language is the one they developed called ReasonML. It’s a dialect of OCaml, which is a dialect of ML.

It touts safety and interoperability with both Javascript and OCaml ecosystems (https://reasonml.github.io/).

Unfortunately, Reason isn’t a Pure Functional Language and so it suffers from many of the problems that Imperative Languages do. It’s a compromise the way that TypeScript is a compromise.

There are a few books on Reason, Web Development with ReasonML: Type-Safe, Functional Programming for JavaScript Developers and ReasonML Quick Start Guide: Build fast and type-safe React applications that leverage the JavaScript and OCaml ecosystems

### Fable

For those married to the .NET ecosystem, there’s Fable, a language that lets you write in Microsoft’s Functional Programming Language, F#, and compiler to Javascript.

Fable supports most of the F# core library and most of the commonly used .NET APIs (https://fable.io/).

Unfortunately, like ReasonML, Fable is not a Pure Functional Programming language either.

I couldn’t seem to find any books on it but here’s a free online “book”, The Elmish Book. The use of the word “Elm” in the title seems to be coincidental and has nothing to do with the Elm language.

### PureScript

![PureScript logo](https://res.cloudinary.com/dehs6irlh/image/upload/v1630256482/jjba-site/blog/breaking-free-from-js/1_w0WBjtEDr_ESj8FdDApaOw_jimozj.png){: .center-image }

PureScript was developed by Haskell developers who stole as much as they could from Haskell while making some great improvements along the way.

This language is my personal favorite. All new projects at my company will be developed in this language. It has all of the expressive power of Haskell yet it runs beautifully in the Browser producing clean readable Javascript (https://www.purescript.org/).

It’s a Pure Functional Language (hence its name) just like Haskell. It is the most powerful of all the languages listed here but unfortunately has a big downside.

The learning curve is pretty steep. That’s what motivated me to write my book to make that process as painless as possible. Once you put in the work to learn PureScript, it will pay you back ten fold in dividends.

There are 2 books I know of. The free one, which is great if you already know Haskell, PureScript by Example.

If you’ve never seen a Functional Language or if you have no idea what Functional Programming is and you’re interested in the most powerful of all of the aforementioned languages, then I’d suggest you consider my book, Functional Programming Made Easier: A Step-by-Step Guide.

It’s a complete Functional Programming course for Imperative Programmers of any language. It starts from the very beginning of Functional Programming and by the end, you’ve developed a web-server and front-end single page application in PureScript.


## Too Good to be True?

![Bear trap](https://res.cloudinary.com/dehs6irlh/image/upload/v1630256482/jjba-site/blog/breaking-free-from-js/1_Hgvz80kAKCqSaMojl5oP1g_iipeq9.png){: .center-image }

All of these languages are very different from Javascript. They can be downright scary. It’s not like going from Java to C# or from Java to Javascript.

These are Functional Programming Languages. You might be thinking that what you’d really like is Java, C# or Python but in the Browser.

But the biggest gains in program safety and developer productivity is only possible from a Functional Programming Language.

That’s a pretty bold claim and for the unconvinced, I invite them to ask programmers who routinely program in a Functional Language in their professional work and ask them if they miss their old Imperative Programming Languages.

I’d be willing to bet that 99 out of 100 would say that they’d never go back to the old way of programming. Then ask them how much effort it took to learn these new fangled beasts.
