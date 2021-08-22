---
layout: post
title: Why Haskell in production as first choice @ Foxdown Systems ?
date: 2021-08-22
---

Haskell is the first programming language we reach for when we build production software systems. This likely seems unusual to anyone who only has a passing familiarity with the language. Haskell has a reputation for being an advanced language with a steep learning curve. It is also often thought of as a research language with limited practical utility.

While Haskell does have a very large surface area, with many concepts and a syntax that will feel unfamiliar to programmers coming from most other languages, it is unrivaled in the combination of developer productivity, code maintainability, software reliability, and performance that it offers. In this post I will cover some of the defining features of Haskell that make it an excellent, industrial-strength language that is well-suited for building commercial software, and why it is usually the first tool we consider using for new projects.
Haskell has a strong static type system that prevents errors and reduces cognitive load

Haskell has a very powerful static type system which serves as a programmer aid that catches and prevents many errors before code ever even runs. Many programmers encounter statically typed languages like Java or C++ and find that the compiler feels like an annoyance. By contrast, Haskell’s static type system, in conjunction with compile-time type checking, acts as an invaluable pair-programming buddy that gives instantaneous feedback during development.

There’s a far smaller cognitive load that needs to be maintained when writing Haskell than when writing in languages like Python, JavaScript, or PHP. Many concerns can be completely offloaded to the compiler rather than needing to be remembered by the programmer. For example, when writing Haskell, there’s no need to preemptively ask questions like:

- Do I need to check whether this field is null?
- What if fields are missing from the request payload?
- Has this string already been decoded to an integer?
- What if this string can’t be decoded to an integer?
- Will this operator implicitly convert this integer to a string?
- Are these two values comparable?

This is not to say that these are questions that never need answering in Haskell; it’s to say that the compiler will throw an error when you need to address one of these issues. For example, it’s possible that a Haskell program needs to handle values that are sometimes not present, but instead of setting any value to NULL, a Haskell programmer must use a Maybe type, which indicates that the value may not be there, and the compiler forces the programmer to explicitly handle the Nothing value; the case where the value is not present.

Haskell’s static type system also leads to other benefits. Haskell code uses type signatures that precede its functions and describe the types of each parameter and return value. For example, a signature like Int -> Int -> Bool indicates that a function takes two integers and returns a boolean value. Since these type signatures are checked and enforced by the compiler, this allows a programmer reading Haskell code to look only at type signatures when getting a sense of what a certain piece of code does. For example, one would not use the type signature above when looking for a function that manipulates strings, decodes JSON, or queries a database.

Type signatures can even be used to search through the entire corpus of Haskell code for a relevant function. Using Hoogle, Haskell’s API search, we can search for a type signature based off of functionality we know that we need. For example, if we need to convert an Int to a Float, we can search Hoogle for Int -> Float (search results), which will point us to the aptly named int2Float function.

Haskell also lets us create polymorphic type signatures through the use of type variables, represented by lowercase type names. For example, a signature of a -> b -> a tells us that that the function takes two parameters of two arbitrary types, and returns a value that whose type is the same as the first parameter. Suppose we want to check whether an element is in a list. We’re looking for a function that takes an item to search for, a list of items, and returns a boolean. We don’t care about the type of the item, so long as the search item and the items in the list are of the same type. So we can search Hoogle for a -> [a] -> Bool (search results), which will point us to the elem function. Parametric types are an extremely powerful feature in Haskell and are what enable writing reusable code.
Haskell enables writing code that is composable, testable, and has predictable side-effects

In addition to being statically typed, Haskell is a pure functional programming language. This is one of Haskell’s defining features and what the language is well known for, even amongst programmers that have only heard of Haskell but never used it. Writing in a pure functional style has many benefits, and is conducive to a well-organized code base.

The word “pure” in “pure functional programming” is significant. Purity in this sense means that the code we write is pure, or free of side-effects. Another term that describes this is referential transparency, or the property where any expression (e.g. a function call with a given list of parameters) can be replaced with its return value without changing the functionality of the code. This is only possible when such pure functions do not have side effects, such as creating files on the host system, running database queries, or making HTTP requests. Haskell’s type system imposes this sort of purity.

So does being pure mean that Haskell programs cannot have side effects? Certainly not—but it does mean that effects are pushed to the edge of our system. Any functions that perform I/O actions (such as querying a database or receiving HTTP requests) must have a return type that captures this. This means that type signatures like the ones we saw in the previous section (e.g. `Int -> Float` or `a -> [a] -> Bool`) are indicators that the corresponding functions do not produce side effects, since `Float` and `Bool` are just primitive return types. For a contrasting example that includes a side effect, a function signature of `FilePath -> IO String` indicates that the function takes a file path and performs an I/O action that returns a string (which is exactly what the `readFile` function does).

Another feature of a pure functional programming paradigm is higher-order functions, which are functions that take functions as parameters. One of the most commonly used higher-order functions is `fmap`, which applies a function to each value in a container (such as a list). For example, we can apply a function named square, which takes an integer and returns that integer multiplied by itself, to a list of integers to turn it into a list of squared integers:

```haskell
square :: Int -> Int
square x = x * x

fmap square [1,2,3,4,5] -- returns [1,4,9,16,25]
```

Code written in this style tends to be both composable and testable. This above example is trivial, but there are many applications of higher-order functions. For example, we can write a function like renderPost which takes a record of post data and returns the version of the post rendered in HTML. If we have a list of posts, we can run fmap renderPost postList to produce a list of rendered posts. Our renderPost function can be used in both the single case and the multi-post case without any changes, because composing it with fmap changes how we can apply it. We can also write tests for the renderPost function and compose it with fmap in our tests when validating the behavior for a list of posts.
Haskell facilitates rapid development, worry-free refactoring, and excellent maintainability

Through the combination of the aforementioned static types and pure functional style that Haskell has, developing software in Haskell tends to be very fast. One of the common development workflows we employ is relies on a tool called ghcid, a simple command line tool that relies on the Haskell REPL to automatically watch code for changes and incrementally recompile. This allows us to see any compiler errors in our code immediately after saving changes to a file. It’s not uncommon for us to open only a terminal with a text editor in one pane and ghcid in another while developing applications in Haskell.

While manually validating the results of our code is eventually necessary, such as by refreshing a page in a browser or using a tool to validate a JSON endpoint, a lot of this can be deferred until the end of a programming session. Many of the runtime errors that a programmer would encounter when writing a web service in a language like Python or PHP are caught immediately and displayed as compiler errors by ghcid. This is a far cry from the need to switch to a browser window and refresh the page after making a change to some code; a development workflow that everyone who has worked on a web application is intimately familiar with.

Beyond the tight feedback loop during development, Haskell code is easy to refactor and modify. Like real world code written in any other language, such code written in Haskell is not write-only. It will eventually need to be maintained, updated, and extended, often by developers that are not the original authors of the code. With the aid of compile-time checking, many code refactors in Haskell become easy; a common refactoring workflow is to make a desired change in one location and then fix one compiler error at a time until the program compiles again. This is far easier than the equivalent changes in dynamically typed languages that offer no such assistance to the programmer.

Proponents of dynamically typed languages will often argue that automated tests supplant the need for compile-time type checking, and can help prevent errors as well. However, tests are not as powerful as type constraints. For tests to be effective, they must:

- Actually be written, yet many real world code bases have limited testing.
- Make correct assertions.
- Be comprehensive (test a variety of inputs) and provide good coverage (test a large portion of the code base).
- Be easy to run and finish quickly, otherwise they will not become part of the development workflow.
- Be updated and maintained in tandem with the code they test.

Haskell’s type system has none of the above issues. The type system is a fixture in the language and the compiler always validates that the types are correct. The type system is inherently comprehensive, providing full coverage of every piece of Haskell code, and there are no changes to make to it as the underlying code changes. All this is not to say that the type system can replace every type of test. But what it does do is provide assurances that are more comprehensive than tests, and are present in every code base, even when no tests exist.
Haskell programs have stellar performance, leading to faster applications and lower hardware costs

GHC, the most commonly used Haskell compiler, produces extremely fast executables, especially when compared against other languages commonly used for application development, such as PHP or Python. This improved performance leads to both a more responsive application and lower hardware costs.

It’s common to hear proponents of other languages be dismissive when their language is described as slow, as hardware is a relatively small cost compared to the cost of hiring programmers. This may be true, but we have found that the difference between Haskell and other languages used for web development is staggering.

On one project we worked on in the past, we began implementing new API endpoints in a Haskell web service instead of the incumbent PHP. After around a year of building features and adding endpoints in Haskell, both the PHP and Haskell web services were dealing with a similar average workload in terms of request count and type, and performed similar CRUD actions backed by the same SQL database. The infrastructure was hosted on AWS, and the breakdown of the infrastructure used for each web service is below.

```
Web Service Language 	EC2 Instance Type 	CPU 	RAM 	Monthly Cost Per Instance 	Number of Instances Used 	Total Monthly Cost
PHP 	c5.xlarge 	4 Dedicated CPU cores 	8 GB 	$122 	2 	$244
Haskell 	t3.nano 	2 Flex CPU cores (limited to 20% use) 	0.5 GB 	$3.75 	4 	$15
```

In this application, each of the Haskell and PHP web services handled a similar number of requests, handled a similar workload, and had similar traffic spikes throughout the day, all while querying the same database. Both the PHP and Haskell web services used Nginx as a reverse proxy. In the end, the cost of operating the Haskell infrastructure was roughly 1/16th (or 6%) of what the PHP infrastructure was. Examining our AWS usage metrics, the CPU on our Haskell machines never even hit 5%. The Haskell endpoints consistently had response times of 100ms or less, slightly outperforming the PHP endpoints.

Ultimately, we had two web services, one written in Haskell and the other written in PHP, that had similar performance but the former had a cost of $200/year and the latter had a cost of $3,000/year. It’s worth noting that the user base of this application was relatively small, with under 25,000 monthly active users (MAUs). This difference in cost would scale as the size of the user base, number of MAUs, and underlying infrastructure increased.

It’s certainly possible to criticize this comparison, and I do not claim that it is in any way scientific. But it’s clear to me that based off of our past experience running production workloads, Haskell outperforms PHP by at least an order of magnitude (and PHP 7.0+ performs remarkably well compared to many other similar languages). The cost reduction that comes with operating Haskell over other web languages is not by any means insignificant.
Haskell is great for domain modeling and preventing errors in domain logic

Another benefit of Haskell’s type system beyond simple compile time type-checking is that it enables modeling a problem domain through the use of custom data types within an application. This allows a programmer to create a description of business logic rules that are enforced by the type system. Haskell has what are referred to as algebraic data types (ADTs), consisting of both records (product types) and tagged unions (sum types). Records are similar to dictionaries or JSON objects, and commonly available in many languages. Tagged unions, however, are not available in many languages, but are what enable a significant amount of flexibility in domain modeling.

The power of ADTs is best illustrated through an example. Suppose we are creating an invoicing system that must keep track of customer invoices. Each invoice must contain a list of line items that the invoice is for and have an invoice status that indicates whether the order has been paid or canceled. The types we would use to model this might look like the following:

```haskell
type Dollars = Int

data CustomerInvoice = CustomerInvoice
    { invoiceNumber :: Int
    , amountDue     :: Dollars
    , tax           :: Dollars
    , billableItems :: [String]
    , status        :: InvoiceStatus
    , createdAt     :: UTCTime
    , dueDate       :: Day
    }

data InvoiceStatus
    = Issued
    | Paid
    | Canceled
```

Modeling domain rules in the type system like this (e.g. the status of an invoice is either Issued, Paid, or Canceled) results in these rules getting enforced at compile time, as described in the earlier section on static typing. This is a much stronger set of guarantees than encoding similar rules in class methods, as one might do in an object oriented language that does not have sum types. With the type above, it becomes impossible to define CustomerInvoice that doesn’t have an amount due, for example. It’s also impossible to define an InvoiceStatus that is anything other than one of the three aforementioned values.

One application of the above types may be a function that creates a notification message based on the status of the invoice. This function would take a CustomerInvoice as a parameter and return a string representing the content of the notification.

```haskell
createCustomerNotification :: CustomerInvoice -> String
createCustomerNotification invoice =
    case status invoice of
        Issued ->
            "Invoice #" ++ show (invoiceNumber invoice) ++ " due on " ++ show (dueDate invoice)

        Paid ->
            "Successfully paid invoice #" ++ show (invoiceNumber invoice)

        Canceled ->
            "Invoice #" ++ show (invoiceNumber invoice) ++ " has been cancelled"

```

The above function uses pattern matching, another feature in the language, to handle every possible InvoiceStatus value. The case statement allows us to handle the different possible values of the status field.

The type system can protect us from making mistakes when changing the rules of our domain. Suppose that after this application is live for a while, we get feedback from our users that we need to be able to refund invoices. To facilitate this, we’ll update our InvoiceStatus type to include a Refunded value constructor:

```haskell
data InvoiceStatus
    = Issued
    | Paid
    | Canceled
    | Refunded
```

If this is the only code we change, then upon compilation, we get the following error:

```
CustomerInvoice.hs:(15,5)-(20,35): error: [-Wincomplete-patterns, -Werror=incomplete-patterns]
    Pattern match(es) are non-exhaustive
    In a case alternative: Patterns not matched: Refunded
   |
15 |     case status invoice of
   |     ^^^^^^^^^^^^^^^^^^^^^^...

```

Whoops! Looks like we forgot to update the createCustomerNotification function to handle this new status value. The compiler is throwing an error and telling us that the case statement does not handle the Refunded value as part of its pattern matches.

By modeling our domain in our types, the compiler assists us in ensuring that all of our domain logic can handle every possible value in the domain*. This protects us from the very common mistake of an unhandled value when writing in dynamically typed languages. Automated tests are not a replacement for types in this situation, because the introduction of new possible values often requires updating tests to assert whether the new values can be handled, which doesn’t help us avoid the problem—it’s just as easy to forget to update tests for the business logic as it is to forget to update the business logic.

* By default, GHC (the Haskell compiler) will not throw an error in the case of an unhandled value, but it’s standard practice for production Haskell projects to use the -Wall and -Werror flags, which turn on nearly every available warning and turn all warnings into errors.

Haskell has a large number of mature, high-quality libraries

The Haskell community has a published a large number of high quality, production grade packages, many of which have been maintained for for a decade or longer. The Haskell community has general consensus as to which packages are good options in each functional category (e.g. decoding/encoding JSON, parsing XML, decoding CSVs, working with SQL databases, HTML templating, websockets, using Redis, etc). In some categories there is a single, best option that is the de facto standard. In other categories, there are several comparable options to choose from, depending on what design decisions or trade offs a developer is willing to make.

Haskell has over 21,000 packages available in its package repository, Hackage, and many more published in various places such as GitHub that build tools can depend on. However, this number is dwarfed by the number of packages available in the repositories of many other languages. As of this post’s publication date, Ruby has 164,000 gems published. There are 282,000 Python packages on PyPI. There were over 1.3 million JavaScript packages on npm as of April 2020.

This discrepancy leads to one of the reservations I have heard expressed about using Haskell in production: there aren’t as many Haskell packages available as there are in other languages. My response to this is that when building production systems, the total number of packages available for a given language is largely irrelevant.

When building a production system, the decision of which packages to use is never based off of the total number of packages available, but which individual packages have a good reputation, widespread use, and other factors such as good documentation and whether a given package is still being maintained. To put it simply, it’s quality and not quantity that matters, and to that end, the Haskell community does an excellent job at curating the packages necessary for real world use cases I described earlier.
Haskell makes it easy to write concurrent programs

One feature of being a pure functional language is that, by default, values in Haskell are immutable. This is not to say that values never change, but state is not changed in-place. For example, when a function appends an element to a list, a new list is returned and the memory used by the old list will be freed by the garbage collector. A benefit of such of immutability is that it simplifies concurrent programming. In a language with mutable values, multiple threads accessing the same value can lead to issues such as race conditions and deadlocks.

Since values in Haskell are immutable, there is no risk of these types of issues even when a program is running on multiple threads and accessing shared memory. This also results in a simpler mental model surrounding concurrent programming. Concurrent code can often be written in the same style as single-threaded code, with functions that run the underlying workload on a new thread simply wrapping the single-threaded implementation.

Concurrency is a useful tool in the Haskell programmer’s toolbox. On projects we have worked on in the past, we have done everything from implemented websocket servers that run as part of the same executable that serves an HTTP API, to created a multi-threaded worker system that required far less overhead than managing individual Linux processes necessary for workers written in languages with limited concurrency support.
Haskell enables domain-specific languages, which foster expressiveness and reduce boilerplate

Haskell’s type system and language features make it a common choice for writing compilers. One offshoot of this is that Haskell libraries sometimes employ domain-specific languages (DSLs) to improve their usability. A DSL, in contrast to a general purpose language, is a small language designed to be well-suited for expressing the rules of a specific application or problem domain.

One of the most well known and widely used DSLs is SQL, which is the language used to query data stored in relational database systems. Unlike most languages, SQL is declarative rather than imperative. This means that a SQL program tends to describe what the outcome of its execution should be rather than how that outcome should be achieved. Any developer familiar with SQL can imagine how writing code to retrieve data stored in tables as a series of rows in an imperative style would be very cumbersome.

One of the features in Haskell that facilitates DSLs is called Template Haskell. This is commonly employed by library authors to allow consumers of the library to use what is an expressive syntax to avoid a lot of boilerplate. One example of this is in the Persistent library, one of the most popular SQL libraries. Persistent exposes a DSL that uses what is referred to as Persistent Entity Syntax that allows the user of the library to define their database schema. An example of this syntax is below.

```haskell
Person
    name Text
    age Int Maybe
BlogPost
    title Text
    authorId PersonId
    publicationDate UTCTime
BlogPostTag
    label Text
    blogPostId BlogPostId
```

The code above is not Haskell, and if you have never used Haskell’s Persistent library, odds are you have never seen this syntax. Yet it is apparent what it does—it defines three tables (Person, BlogPost, and BlogPostTag) and the columns within them. This code gets consumed by a Haskell program and supplants the need to write approximately 150 lines of Haskell code to define all of the data types and accessor functions for working with the data from these three tables.

The above is only one example of an external DSL, which is a DSL that uses its own syntax. Other libraries that expose DSLs include ones for webserver route definitions and for HTML templating. Some library authors opt to create embedded domain-specific languages (eDSLs), which are written in Haskell syntax. This results in a series of types and functions that are specialized to a particular domain. Esqueleto is an example of a widely-used library that exposes an eDSL for writing type-safe SQL queries.
Haskell has a large community filled with smart and friendly people

One of the most important facets of using a programming language is the community. Haskell’s community is large and includes a wide variety of people coming from many different technical backgrounds. This includes programming language researchers, some of whom have been working on Haskell since its inception in 1990, creators of other programming languages whose compilers are written in Haskell, self-taught Haskell enthusiasts, professional Haskell programmers using Haskell commercially (we at Foxhound Systems fall into this category), as well as eager-to-learn students, amongst many others.

The Haskell community is very welcoming to beginners. While the language has a learning curve that is steeper than that of many others due to its depth and breadth, it’s easy to ask questions and find help any number of people that sincerely want to help others learn the language.

Some of the forms of communication we like to use to engage with the Haskell community are:

- The Haskell subreddit, which has over 60,000 readers and is one of the largest programming language communities on reddit.
- The Functional Programming Slack, which has a number of channels dedicated to Haskell (including #haskell, #haskell-beginners, #haskell-jobs, and #haskell-adoption).
- The Haskell mailing lists, such as haskell-cafe, which have a variety of content from library announcements, to Q&A about the language, to volunteer opportunities
- The #haskell channel on the Freenode IRC network often has over 1,000 people connected to it, and is a great alternative to the Slack channels.
- The Haskell Weekly Newsletter, which is a weekly newsletter that highlights blog posts and other announcements from the preceding week.
- Although not conventional community, the haskell tag on StackOverflow has over 46,000 questions associated with it. It’s not uncommon to find excellent answers that give a great overview of a specific topic or issue related to the language.

This is not an exhaustive list, and participation through every forum is not necessary. But when someone is looking for help or generally learning about the language, it’s worth using any of the forums above.
Conclusion

There are many reasons for why Haskell is our first choice of programming language for building production software systems. To recap the whole list covered in this post:

- Haskell has a strong static type system that prevents errors and reduces cognitive load.
- Haskell enables writing code that is composable, testable, and has predictable side-effects.
- Haskell facilitates rapid development, worry-free refactoring, and excellent maintainability.
- Haskell programs have stellar performance, leading to faster applications and lower hardware costs.
- Haskell is great for domain modeling and preventing errors in domain logic.
- Haskell has a large number of mature, high-quality libraries.
- Haskell makes it easy to write concurrent programs.
- Haskell enables domain-specific languages, which foster expressiveness and reduce boilerplate.
- Haskell has a large community filled with smart and friendly people.

It is the sum of these reasons that makes Haskell such a compelling choice. Haskell enables rapid development, worry-free refactoring, easy maintainability, provides excellent performance, and has a mature ecosystem. These facets among many others make it an excellent choice for building production applications.

Original post [here](https://www.foxhound.systems/blog/why-haskell-for-production/)

