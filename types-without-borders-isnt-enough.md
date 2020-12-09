At Elm Conf 2018, I gave a talk called Types Without Borders. In the talk, I discussed the idea of preserving type information when crossing the boundary between languages or environments.

I gave a demo of two different libraries that follow that principle: elm-graphql, and elm-typescript-interop. elm-graphql has stood the test of time quite well.

elm-typescript-interop was a solid idea at the time, but it missed something fundamentally that elm-graphql got right. So I'm rethinking it from scratch and introducing a new incarnation of that idea that I'm calling elm-ts-interop.

## The original elm-typescript-interop

The idea of elm-typescript-interop is that it looks at your Elm source code and finds all the ports with their type information. In Elm, a port is a way to send messages to or from JavaScript. Since JavaScript doesn't have the same guarantees that Elm does, ports are a way to have an environment with sound types and no runtime exceptions. But you can still communicate with JavaScript, just by sending messages with the concept of the Actor Model - messages go out, messages come back. They are asyncronous.

You can annotate your ports with type information, like this:

showModal : { title : String, message : String, style : String } -> Cmd msg

And in your app, you would call

showModal { title = "Could not find that discount code", message = "Could not found discount code " ++ discountCode ++ ". Please try again.", style = Warning |> styleToString }

elm-typescript-interop takes that type information and generates TypeScript type definitions for setting up your Elm code. This is quite handy because it gives you

- Intellisense for autocompletions when you wire up your ports in your JavaScript entrypoint
- TypeScript errors if you pass in incorrect data
- TypeScript type information about the incoming data from Elm

Wiring up your ports looks something like this:

```js
const app = Elm.Main.init({flags: flagData})

app.ports.showModal.subscribe(...)

```

This all works as expected. In hindsight, this gives you type-safety, but it misses the bigger picture.

## The problem with the original approach

People would often ask, "what's the best way to send your Elm Custom Types through a port?"

Initially, I thought that I would eventually come up with a way to automatically serialize Elm types, again by statically analyzing your Elm source code. Then you could serialize that data into TypeScript types automatically. I tried sketching out some design ideas, and never came up with anything that felt satisfying.

So what that left you with was using elm-typescript-interop to be a serialization/de-serialization layer. But you would then need a second layer to convert that into proper Elm data.

TODO - example of converting a nice Elm custom type into automagically serializable Elm data.

It needs a convert function.
