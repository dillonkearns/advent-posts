# The Power of Fractal Transformations

One of my favorite things about functional programming is the ability to work and think in a very localized area of code. Let's talk about some of the patterns and tools that make that possible.

I won't go into how immutability, managed effects, and no global variables/environment helps us think more locally - although it really does! What I want to focus on here is how one little tool helps in terms of individual transformations composed together, rather than one big transformation: the map function.

## Thinking In Fractals Vs. Monoliths

In my JavaScript days, I remember many times when I was mapping over a big JS object, writing unit tests for a giant set of changes all made at once. I remember a specific case where we had a big list of data in the format defined by the server. We needed to take these products, pull out specific pieces of data, normalize them, and then apply the filters from the UI to show/hide certain items, and to sort them based on sorting options.

There were many nested fields, data could be null or not, and the options from one product line or another needed to be normalized differently.

We had a lot of big unit tests to make sure things were working. Even so, it was so difficult to go in to our series of lodash function calls and find _where_ you needed to make a change. And once you did, you would want to make sure you added several new test cases to make sure you didn't miss a spot or mishandle a special case.

The challenge was that using that paradigm to normalize JSON data required thinking of the data as a monolith. We certainly abstracted out functions to help with parts of the normalization. And we used lodash to do functional style mapping over the arrays of data and key-value objects. But mapping over arrays and objects only gets you part of the way there. We still needed to keep a map in our heads of the structure of the data so we could go into a specific area, traverse it, and change it.

A Monolothic Transformation has you thinking top-down. With a Fractal Transformation, you think bottom-up.

Working with this code in a functional language like Elm opens up a totally different paraidigm for doing these transformations. Using a JSON Decoder, we can flip this big transform function on its head and turn it into a series of small, localized transforms. Let's say, for example, that we need to filter out products that aren't within the specified price

##

The most common example of mapping is mapping over a list. We have a List of User values, and we need to turn it into a List of user names. List.map User.userName users. This is great! But that's only scratching the surface.

One of my favorite examples of this pattern in Elm is JSON Decoders. This is one of those things that you hate the first time you see it (what, I have to write out everything I expect in my JSON explicitly!?). But then once you get over the hump, you can't imagine life without it.

In my JavaScript days, I remember many times when I was mapping over a big JS object, writing unit tests for a giant set of changes all made at once. I remember a specific case where we had a big list of data, and that format had of course come back from the server, sometimes in a somewhat strange format. We needed to take these products, pull out specific pieces of data, turn it into the right format, and then apply the filters from the UI to show/hide certain items, and to sort them based on sorting options.

That's a mouth full to just explain in words. The code was even harder to follow. There were many nested fields. You weren't sure when they could be null or not, when the server may return a slightly different format for the options in one product line or another, etc.

We had a big test case for this, but even so it was so difficult to go in to our series of lodash function calls that it was hard to be confident about even finding _where_ you needed to make a change.

Working with this code in a functional language like Elm opens up a totally different paraidigm for doing these transformations. Using a JSON Decoder, we can flip this big transform function on its head and turn it into a series of small, localized transforms. Let's say, for example, that we need to filter out products that aren't within the specified price

ProductOptions.elm

decoder =
Decode.map5
Decode.field "color" colorDecoder ...

If we need to change the logic around so that we are accounting for a new filtering option, we can do that in one place. We don't need to go into a big transform function

transform(productCatalog: Product[][]): Product[][]

And use some lodash magic to find exactly the right insertion point, and carefully transform to preserve the types. We have a single pinch point that we can modify to make that localized incision.

The same pattern applies to many different paradigms. And we don't just need to be dealing with JSON to apply these localized maps. It's just snapping together these lego blocks of various transforms to describe a bigger transformation, and finding and working on that change is so much easier. If we get some really tricky logic in one spot, like decoding the colors for different types of products, then we can write a unit test for that specific part of the chain. Or not! It's very easy to change. With our lodash version, it would be difficult to carve out that small piece of it into an individually tested local transformation.

And we can use this same localized transformation pattern with many different patterns: elm-graphql, Maybe, List, etc.

So what's the secret? Conceptually (and in fact under the hood), a JSON Decoder represents a function. It takes a JSON value, and returns some transformed data (or an Error if the shape of the JSON didn't match the decoder).

So we can map into any point in those transformations. But this function is special, because not only can you pass it around and apply it, but you can chain on a transformation and build up changes to that part of the pipeline.

Imagine a music production command center where you can pull in various beats and tracks, and transform each of them individually. You can speed up all the tracks, or you can just have the drum beat go twice as fast. You can apply an effect to just the bass line. The power of it is that you don't have to deal with it as a whole, you can work with it at any level.

How to use this to have localized, ENCAPSULATED reasoning - moving some local transforms into an opaque module.

## How do you design for Fractal Transformations?

It sounds complicated and hard to wrap your brain around, but once it clicks it's easier to think about. Much like thinking about recursion. "Define a function in terms of itself" sounds intimidating. Until you realize it's quite natural.

```elm
fibonacci n =
  if n <= 1 then
    n
  else
    fibonacci ( n - 1 ) + fibonacci ( n - 1 )
```

Fractal transformations are a similar concept. A transformation defined as either a core transformation, or a composition of transformations.

What is an Elm JSON Decoder? It's either one of the core primitives (a String decoder, Int decoder, etc.), or it's a composition of the two.

Want to change the data type of part of it? Instead of reaching in to a list of lists within fields of a particular name in a JSON object (imperative), we can tweak the relevant decoder that we've composed and make the change locally.

```elm
colorDecoder
```

## Type-Safety Isn't Enough

We can use that pattern in different languages, and often functional-style languages pull in type systems but omit the power of fractal transformations.
