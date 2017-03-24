> ### Now Playing: Exercise 5 (02:05:18 - 02:09:29)

This example looks very similar to the last one, with a few additions:

- `Msg` has changed, and its values (constructors) are now functions which take additional arguments.
- Instead of having `type alias Msg`, it's now `type Msg` -- which is to say that it's now a **Union Type** instead of a Record.

Our goal with the first exercise is to start recording the `SetQuery` message. In a future exercise, we'll extract that value from the Model and use it in a query to GitHub's API, but to do that we need to first get it _into_ the model.

### Step 1 -- write it down

Typically the best practice for doing this is to use the `OnInput` function. You could also use `OnKeyDown`--but `OnInput` is a little bit better because it also captures things like copy and pasting with mouse events. `OnInput` basically captures **all** of the ways that you might manipulate a text field.

So, we'll use `OnInput` and define its behavior in a way to construct messages and send them to event handlers that we'll write here in a bit. The point being that we need to somehow get this user input data into the model.

> Something to know about `OnInput`. When you call it, you should only pass **ONE** argument, and that argument is supposed to be a _function_ which takes a string and returns some value (often a `message`).

It's worth noting that we already have a few functions which take `String` or `Int` arguments and return `Msg`'s...

- `DeleteById` is a function which takes an id and returns a message
- `SetQuery` is a function which takes a string and returns a message

(These may be helpful when working on the first _TODO:_)

It's essentially the same idea as before, except instead of passing a record, ie.

```elm
-- old way
view model =
    ...
        ...
            , input
                [ ... ]
                onInput (\str -> { operation = "SET_QUERY", data = "blah" })
```

We'll now pass our union type and decide behavior in our update function based upon the type of data it receives.

#### Note:

Where the `OnInput` handler takes a function that will generate the message, `OnClick` just takes the message itself. So when we call it, we'll just pass an individual message rather than a function.

Finally, we need to update our `update` function to account for the new  constructors present in `Msg`. So it's the basically the same as before, only now instead of using an If expression, we're going to use a Case expression to pull apart the different pieces of our Union Type and do the right thing.

> #### Now Playing: Audience Questions (02:09:29 - 02:13:03)

##### Question: Is the constructors in these unit types are function, where is the body of these functions? Where does it say what they do?

There is no "body" per se. These are "logic-less" functions, their entire purpose is to build up these "containers" of values. If we go to the REPL...

```elm
> type Msg = SetQuery String | DeleteById Int
> SetQuery
<function> : String -> Msg
> DeleteById
<function> : Int -> Msg
```

A `Msg` is sort of a _container_ that holds "two different flavors" of value. One is a string **wrapped** in a `SetQuery`, and the other is a an integer **wrapped** in a `DeleteById`.

The only way to get at these values is to write a Case expression:

```elm
...
    case msg of
        SetQuery query ->
            -- do setting of query stuff
        ...
```

One of the things that you can _**rely on**_ is that **anytime** you see a function with an UPPERCASE name, there is _NO_ logic in it. It's just for labeling/wrapping/returning these values.


> ### Now Playing: Exercise Solutions (02:13:03 - 02:21:48)

#### Question: How to you check that the update function is working and the data you're passing is correct?

There is at least one simple, quick-and-dirty way to accomplish this which is to simply take that data and throw onto the screen, within a `text` function.

Another, and safer way to is use `Debug.log`. This will output stack traces and other info to the console.

##### How does `Debug.log` work?

Because _everything_ in Elm is an **expression**, `Debug.log` has to return something. And, what it returns is _the thing that you give it_. So we effectively wrap some value in a `Debug.log` expression.


> ### Now Playing: Audience Questions (02:21:48 - 02:24:12)

#### Question: When you call `DeleteById`, is it "executing" that right away?

Yes.. but it's important note the type of `DeleteById` itself, namely that its a function/value that _just_ returns a Msg. It doesn't actually "delete", we had to write the test function to pass to `List.filter` along with the list of results.

This may be generally true of constructors within union types, they take values and return message wrapped in those values.

## A note on `whitespace`

For the most part, Elm® does not care about how you use whitespace, with a small number of crucial exception, the first being Case Expressions, which encode a rule that all the branches of your expression should start at the same level of indentation. The following is invalid Elm code:

```elm
update msg model =
    case msg of
        SetQuery query ->
                { model | ... }
            DeleteById id ->
                { model | ... }
```

This is also true of Let blocks. Here's the good vs. bad indention according to our compiler:

```elm

-- n'uh uh
exampleFn val =
    let
      foo =
            1

        bar =
            2


-- ye'ah huh
exampleFn val =
    let
        foo =
            1

        bar =
            2

```


> ### Good news, though! Other than that, nowhere else in Elm does indentation matter.


> Now Playing: Result and Maybe -- (02:2412 - 02:31-09)

## Result and Maybe

Q: What we have Static data that represents real GitHub data, but what are we missing?
A: The ability to actually talk to GitHub's API and get some _real_ results.

There are two tasks which will go into that.

1. Dealing with the big blog of JSON that we're going to get back
2. Making the HTTP requests

We're going to cover both of those, in that order.

[INSERT IMAGE: "Exercise: resolve the TODOs in part5/Main.elm"]

### JSON Decoding

This will get us from blobs of JSON to **something we can work with**. Let's start with a familiar function to JavaScript, `parseInt()`.

```js
parseInt("42")  // 42
```

What's going on? If we pass a string containing valid numerical characters, parseInt will return actual numbers that correspond to those characters. If we pass it something else, say . .

```js
parseInt("Halibut")
```

JavaScript will return `NaN` -- which I guess is factually accurate. But not we're back into NaN poisoning, which remember is the way that NaN will propagate through computations on sets of numbers which contain a NaN value somewhere.

Elm has a similar function called `String.toInt`, it's sort of like a drop in replacement for parseInt(). But, it works a little bit differently.

```elm
String.toInt "42"
--> Ok 42
```

Notice that we do not get back the _number_ `42`, but we get a particular response `Ok 42`.

`Ok` is a type constructor in a Union Type called `Result`, which holds two values, it's defined (basically) like so,

```elm
type Result =
    Ok somethingGood
    Err somethingBad
```

When we call `String.toInt`, we get back a value AND a message.

```elm
-- REPL
$ String.toInt "42"
>> Ok 42 : Result.Result String Int

$ String.toInt "halibut"
>> Err "could not convert string 'halibut' to an Int" : Result.Result String Int
```

Notice that these can contain different types. (***important*** )

When valid, we get back `Ok` and an Int value, when the string we ask to be parsed is invalid the compiler returns `Err` followed by a string that tells us what went wrong.

This is Elm's primary paradigm for implementing error handling.

What this means is that when you call `String.toInt` on something, you cannot just use the return value right away, what you must do is _first_ run a Case Expression on the return value. In other words, you _have to_ handle the error. Error Handling then becomes something impossible to forget. If there is even the _possibility_ of failure, you're going to get a container with something like `Result`.

### `Maybe`

A "simplified" version of `Result`, in a way. In essence they are the same, but without the error message. It's type signature is,

```elm
type Maybe =
    Just someValue
    Nothing
```

If we think of functions that return `null` or `undefined` in JavaScript, then `Maybe` is maybe (lol) the closest analogue in Elm, whereas `Result` is a bit more like `try/catch`. Where you can _attempt_ something, and if it works proceed with regular execution, otherwise handle the error in some way.

> It's like a container that holds at most one value."

Here's an example of `Maybe` in action:

```elm
-- REPL
$ List.head [ 5, 10, 15 ]
>> Just 5 : Maybe.Maybe number
```

If we call on a non-empty list, we get `Just` and the value of that item. But, if we call `List.head` on `[]`, we get back `Nothing`.

To compare this again to JavaScript, it we try to called `someArray[4]` and there is no 4th element in the array, the interpreter will return `undefined` -- this can be bad as we'll now have some code that returns a value which might not be handled. Again, in Elm that's not possible.

## Pipelines
> 02:31:09 - 02:40:11

Let's look at this code sample:

```elm
List.filer (\num -> num < 5) [ 2, 4, 6, 8 ]
```

In English we might just say, "Filter out all the values that are less than 5" -- or "Only keep the numbers that are greater than 5" -- this is of course ridiculous, it's just fun to attempt to pronounce computer programs into common parlance, at times... :]

Let's say that we then wanted to extend this tiny program, and reverse the list which is returned from the filter example. One way to do it is to just expand the expression, like so

```elm
List.reverse (List.filter (\num -> num < 5) [ 2, 4, 6, 8 ])
```

Just adding this additional function call adds quite a bit of linear logic to our code, so the meaning might not be immediately reachable by a quick scan. A popular way to refactor this is Elm is to use a **pipeline**.

Here's an example:

```elm
[ 2, 4, 6, 8 ]
    |> List.filter (\num -> num < 5)
    |> List.reverse
```

How this works is to essentially perform the following, step-wise:

1. Give me _something_ that takes one argument
2. Call it, passing whatever was return from the previous step in the pipeline

Pipelines are easier to expand upon.

```elm
[ 2, 4, 6, 8 ]
    |> List.filter (\num -> num < 5)
    |> List.reverse
    |> List.map negate
    |> List.head
```

Trying to write all of that as a single expression would obviously get convoluted. Instead, we can write the steps as a pipeline of functions and get a much clearer picture of what's happening.

This is simply known as "Pipeline Style" and one thing to note about it is that they are essentially _free_, meaning that Elm's compiler will do the work of unrolling all of the piped calls into the lengthy expressions that we're trying to avoid, for human-readability purposes. Ultimately, they get compiled to exactly the same JavaScript.

#### Audience Questions

> In Go-lang, error handling like `Maybe` would require a lot of boilerplate--is there a similar cost in Elm?

Since with Elm not every/any value can be `null` or `nil`, you only have to handling errors _wherever an error is possible_. So, "defensive error handling" in Elm, doesn't really exist. Subjectively, I don't feel like it's a burden to handle errors in cases where they **might** arise, and to help that, the Elm ecosystem has a lot of helper functions to make engineering those portions of your code fairly quick and declarative.

A good example of this is `Maybe.withDefault`:

```elm
$ List.head [ 1, 2, 3 ]
>> Just 1

$ List.head [ ]
>> Nothing

$ Maybe.withDefault 42 (List.head [ 1, 2, 3 ])
>> 1

$ Maybe.withDefault 42 (List.head [ ])
>> 42
```

So you're only one function call away from having a very clean way of handling functions which return `Maybe` values. There's some other examples like `Maybe.map` which would handle only the `Just` value and leave the `Nothing` alone.

These helper functions make it so that you don't have to write out the Case every single time.


## Now playing: **Decoders**
> (02:40:11 - 02:50:12)

Or, where we try to _get at_ the values contained within JSON data structures.

```elm
$ decodeString float "123.45"
>> Ok 123.45 : Result.Result String Float

$ float
>> <decoder> : Json.Decode.Decoder Float
```

The first argument is a means of specifying what we expect that the JSON will look like. It is not a function, but rather a label. Its type is simply `Decoder`. Decoders are a kind of opaque type.

Decoders are _composable_.

In this next example, `list` is not a Decoder, but rather a function which **returns** a Decoder of the type which we pass to it:

```elm
-- REPL

$ decodeString (list int) "[1, 2, 3]"
>> Ok [ 1, 2, 3 ]


-- non-REPL, using a pipeline

"[1, 2, 3]"
    |> decodeString (list int)
-- Ok [ 1, 2, 3 ]
```

#### Decoding Objects

Let's set up a hypothetical scenario where we have some kind of game, with some kind of score and some sort of Bool about the state of the game.


```elm
makeGameState score playing =
    { score = score, playing = playing }

decoder = ???

decodeString decoder """{"score" : 5.5, "playing": true}"""
```

How do we write this decoder?

There's a way to do it using **pipelines** again:

```elm
decoder =
    decode makeGameState
        |> required "score" float
        |> required "playing" bool
```

The decoder's responsibility is defining the form of the object that we want to end up with. So `makeGameState` returns the record that we want, and then for each argument that the function takes, we'll perform the pipeline of operations.

"Required" is a means of checking that the value is present and of the type that we expect.

The **order** of "pipes" and the order of arguments to the decode function will match.

From that we can derive that the number of arguments and the number of steps in the pipeline **must _match_**. But don't worry, if you don't provide all of the arguments or miss a step in the pipeline, you'll get an error from the compiler.

#### There is a shorthand, though . .

We talked earlier about Type Aliases, but one thing which was only hinted at is that fact that these aliases also provide the us (for free) a function with a particular signature which can be used in our program. Here's a type alias for `GameState`:

```elm
type alias GameState =
    { score : Float
    , playing : Bool
    }

-- GameState : Float -> Bool -> GameState
```

When working with JSON, a lot of times you might find it necessary to write a bunch of boilerplate which has to do with values being populated for the correct keys in a new object/record somewhere. For example:

```elm
makeGameState : Float -> Bool -> GameState
makeGameState score playing =
    { score = score
    , playing = playing
    }
```

But notice that this function has _exactly_ the same signature as our aliased type, `GameState`--so we can just use that structure directly, skipping the boilerplate construction.

Type Aliases are **callable**--anytime you see an UPPERCASE function in Elm, remember that it's not doing any kind of calculation, it has no function body: it's just returning some data. This is the one other instance in which we have Capitalized functions, namely type aliases for Records.

So, based on the premises above, we can surmise..

```elm
makeGameState 2.3 True == GameState 2.3 True
```

This will come in handy later. Here's our decoder re-written to use the type alias rather than the `makeGameState` function. It's a fairly trivial change, but eliminates an entire class of code duplication.

```elm
type alias GameState =
    { score : Float
    , playing : Bool
    }

decoder =
    decode GameState
        |> required "score" float
        |> required "playing" bool

decodeString decoder
    """{"score": 5.5, "playing" : true}"""
-- Ok { score = 5.5, playing = True}
```

### Questions about Decoders
> (02:50:12 - 02:57:15)

> ### Q1. What happens if you're reading `GameState` from an external API which makes a naming change from 'score' to 'gameScore'?

A1. You would _only_ have to change the part of the decoder function where that field is named, so for instance `|> required "score" float` you would rename `|> required "gameScore" float`. You wouldn't have to make that change anywhere else, unless maybe if there were tests which also required this name field. Otherwise, your internal implementation is completely decoupled from the external API.

> ### Q2. How would you build up the object decoders without using Pipelining?

A2. There are a couple of ways. One way is to use an alternate API, using `Json.Encode.object` and . There's actually a service which will write the decoders and type aliases for you called [JSON-to-Elm](https://eeue56.github.io/json-to-elm). Here's some example output from the JSON object, `{"score": 5.5, "playing" : true}`:

```elm
-- setting: "original"

import Json.Encode
import Json.Decode exposing (field)

type alias Something =
    { score : Float
    , playing : Bool
    }

decodeSomething : Json.Decode.Decoder Something
decodeSomething =
    Json.Decode.map2 Something
        (field "score" Json.Decode.float)
        (field "playing" Json.Decode.bool)

encodeSomething : Something -> Json.Encode.Value
encodeSomething record =
    Json.Encode.object
        [ ("score",  Json.Encode.float <| record.score)
        , ("playing",  Json.Encode.bool <| record.playing)
        ]


-- setting: "pipeline"
import Json.Encode
import Json.Decode
import Json.Decode.Pipeline

type alias Something =
    { score : Float
    , playing : Bool
    }

decodeSomething : Json.Decode.Decoder Something
decodeSomething =
    Json.Decode.Pipeline.decode Something
        |> Json.Decode.Pipeline.required "score" (Json.Decode.float)
        |> Json.Decode.Pipeline.required "playing" (Json.Decode.bool)

encodeSomething : Something -> Json.Encode.Value
encodeSomething record =
    Json.Encode.object
        [ ("score",  Json.Encode.float <| record.score)
        , ("playing",  Json.Encode.bool <| record.playing)
        ]


-- setting: "English"
import Json.Encode
import Json.Decode exposing (field)

type alias Something =
    { score : Float
    , playing : Bool
    }

decodeSomething : Json.Decode.Decoder Something
decodeSomething =
    Json.Decode.map2 Something
        (field "score" Json.Decode.float)
        (field "playing" Json.Decode.bool)

encodeSomething : Something -> Json.Encode.Value
encodeSomething record =
    Json.Encode.object
        [ ("score",  Json.Encode.float <| record.score)
        , ("playing",  Json.Encode.bool <| record.playing)
        ]
{-
If successful, create a record of the type Something
It must have a field called `score` with the type Float
It must have a field called `playing` with the type Bool
-}
```

Notice the use of `map2`, these is a map function for dealing with 2 fields. Likewise there are `object2`, `object3`, `map5`, etc.


> ### Q3. How would you decode a heterogeneous list?

A3. Let's say you have a JavaScript array such as `[2, 4, 6, "hamburger"]`--totally valid JS, not valid Elm. What you can do is write a _custom_ decoder to solve this. This type is called `OneOf`.

```elm
$ oneOf [ intDecoder, stringDecoder]
```

You would give it a list of decoders, but you can't _literally_ do this, because of the inherent type mismatch. What you can do however is create a wrapper type which takes any type of argument and gives them a container function so that the items within the list are now all of the same type.

```elm
type IntOrString = ItIsAnInt Int | ItIsAString String

-- REPL
[ ItIsAnInt 3, ItIsAString "foo" ]
-- [ItIsAnInt 3,ItIsAString "foo"] : List Repl.IntOrString
```

Notice that the list now contains only the type `IntOrString`. Brilliant!

We can now attempt to handle these values with the decoder.

```elm
oneOf [ intToIntOrString, stringToIntOrString ]
```
