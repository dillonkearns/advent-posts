# Combinators - Inverting Top-Down Monolothic Transforms

One of my favorite things about functional programming is the ability to work and think in a very localized area of code. Let's talk about some of the patterns that make that possible.

I won't go into how immutability, managed effects, or lack of global variables help us reason locally - although they really do! What I want to focus on here is the power of individual transformations composed together, rather than making one big transformation. This concept is sometimes called a Combinator.

## What is a Combinator?

The term Combinator is used because you can _combine_ the smaller units to build up the whole. A Combinator is the idea of building something up by combining small "base" values into more complex ones.

It sounds complicated and hard to wrap your brain around, but once it clicks it feels natural. Much like thinking about recursion. "Define a function in terms of itself" sounds intimidating. Until you realize it's very declarative and readable to express things that way.

```elm
fibonacci n =
  if n <= 1 then
    n
  else
    fibonacci ( n - 1 ) + fibonacci ( n - 2 )
```

This recursive definition can be read as _what_ a fibonacci number is (declarative), instead of _how_ it is calculated (imperative). It's pretty close to how you would teach fibonacci to a human. It just so happens that computers can understand it, too!

Recursion is a good analogy for Combinators:

- A Combinator is declarative (not imperative)
- A Combinator is either defined in terms of other Combinators (analagous to a recursive self-invocation), or it is a "base" combinator (analagous to a recursive base case)

Let's look at an example of a Combinator in Elm. The `elm/json` package is how you turn untyped JSON data into typed Elm data (or an `Err` `Result` value if the structure doesn't match).

```elm
personDecoder : Decoder { name : String, birthday : Posix }
    Decode.map2
        (\name birthday -> { name = name, birthday = birthday })
        nameDecoder
        birthdayDecoder
```

What are `nameDecoder` and `birthdayDecoder`? Some kind of decoder. We're _combining_ them.

But at some point, we need to refer to a Decoder that can directly resolve to a value (similar to our recursive "base case").

```elm
nameDecoder =
  Decode.field "full-name" Decode.string
```

`Decode.string` is going to resolve to some value. But we can compose Decoders together in more ways than just combining them or reading JSON values within a JSON property (like `"full-name"`).

Another key technique for a Combinator is that we can transform them.

```elm
birthdayDecoder : Decoder Time.Posix
birthdayDecoder =
  Decode.field "birthday-date-time" iso8601DateDecoder
```

If we follow the definitions, we finally end up with a direct "base" Decoder for `birthdayDecoder`.

```elm
iso8601DateDecoder : Decoder Time.Posix
iso8601DateDecoder =
    Decode.string
        |> Decode.andThen (\dateTimeString ->
        case iso8601StringToTime dateTimeString of
            Ok time -> Decode.succeed time
            Err error -> Decode.fail error
        )
```

This time, we are transforming the raw String into an Elm `Time.Posix` value.

This feels like magic at first, much like recursion does the first time you encounter it. But once you get used to it, it becomes quite natural to define things declaratively this way. And there are some huge benefits to this approach when it comes to narrowing the scope of what you need to pull into your head to understand a section of code or make a change. In other words, Combinators help localized reasoning.

## Top-Down Vs. Bottom-Up Transformations

In my JavaScript days, I remember a particularly tricky area of code where we had a big list of data in a format defined by the server. We needed to take product listings, pull out specific options and inventory information, normalize them, and then apply filters from the UI to show/hide and sort search results.

There were deeply nested fields, and some incongruities in the shape of the data. Some values were nullable. Some had specific normalization we needed to apply to get data from multiple sources to match.

We had a lot of big unit tests to make sure things were working. Even so, it was so difficult to go in to our series of lodash function calls and find _where_ you needed to make a change. And once you did, you would want to make sure you added several new test cases to make sure you didn't miss a spot or mishandle a special case.

The challenge was that using that paradigm to normalize JSON data required thinking of the data as a monolith. We certainly abstracted out functions to help with parts of the normalization. And we used lodash to do functional style mapping over the arrays of data and key-value objects. But mapping over arrays and objects only gets you part of the way there. We still needed to keep a map in our heads of the structure of the data so we could go into a specific area, traverse it, and change it.

We would sit in awe as the person most familiar with the codebase correctly traversed the nested arrays and objects to get to the exact right place and make a change on the first try. It was a lot to hold in our heads, and it was quite error prone.

We weren't using TypeScript at the time, but even if we had been, the challenge would remain of having to navigate the structure from the top down in order to add a new transformation.

These top-down transformations require you to think of the data as a Monolith. With a Combinators, you work bottom-up, and you can think locally.

## Inverting the Monolithic Top-Down Approach

Working with this code in a functional language like Elm opens up a totally different paraidigm for doing these transformations. Using a JSON Decoder, we can flip this big transform function on its head and turn it into a series of small, localized transforms.

Let's say we are getting back some data from the server that we need to normalize. With the top-down, monolothic transformation, the first question is how to find the right data in our data structure. Once we've figured that out, we need to reach in and transform it in all the right places. If the data lives in a deeply nested field inside of an array of arrays inside of an object, then this can get quite tricky. We may be lucky and already have a function that is transforming some data in that area of the code, which we can then piggy back onto. Or it may not quite fit, in which case we need to find a new seam, transforming the data at every level starting from the top.

Let's compare that to a more functional, bottom-up approach.

Remember that our JSON Decoders are using the Combinator pattern, and a Combinator is composed of lots of smaller Combinators. Also, since it is a Combinator, it is declarative. So we don't have to think about _how_ it's transformed, just _what_ transformations are done. That means that we can go in and find the right insertion point.

```elm
fluffyColorOption : Jdec.Decoder FluffyColorOption
fluffyColorOption =
    Jpipe.decode FluffyColorOption
        |> Jpipe.required "b" Jdec.float
        |> Jpipe.required "g" Jdec.float
        |> Jpipe.required "r" Jdec.float
```

This may seem like a small difference. Couldn't we just make this change by tweaking a function for normalizing that data, or creating a function in the right place?

You can, but

- You need to work your way down from the top first
- These top-down changes tend to happen in multiple phases
- Bottom-up transformations give you a frame where you have freedom to think locally

## Combinators Beyond JSON Decoders

This pattern isn't specific to a language, like Elm, or a domain, like JSON.

You can use these same concepts in TypeScript with a library like [`io-ts`](https://github.com/gcanti/io-ts). And you can apply this thinking to a lot more problems than JSON. Some examples in the Elm ecosystem:

- Random number generators
- elm/parser
- Creating fuzzers (property-based test data)
- elm-graphql

## Next Post

Thanks for reading! In the next post, I will share lessons learned about this technique and how it made me realize why I loved one type-safe library I built, and am now rebuilding another to learn these lessons a few years later.
