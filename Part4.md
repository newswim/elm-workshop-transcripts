### If Statements

### Case Expressions

Unlike `case` in JavaScript, `case` expressions in Elm do not "fall through"--meaning they don't have the potential to evoke multiple logic branches--thus there's no need for the usage of the `break` keyword. This has always been a [contentious thing] in javascript, and Elm removes the awkwardness around it.

Rather than an explicit `default` keyword, Elm takes the last clause as its default case, indicated with the `_` underscore character.

It also has an `of` keyword to mark the beginning of a list of possible cases. Take the example from our codebase:


```elm
case msg.operation of
    "DELETE_BY_ID" ->
        -- remove from model
    "LOAD_RESULTS" ->
        -- load more results
    _ ->
        -- default branch
```

### Union Types

A way to specify a type which may take multiple types as its values.

```elm
type Sorting =
    Ascending | Descending | Randomized
```

Let's say hypothetically that we're making a sortable table, and let's say that there are _three_ possible ways that we can sort that table.

In JavaScript, we might have three different Strings that one of a function's formal arguments might be, then within the function body we would check the value of that string, and make branch-wise actions accordingly.

This is a little bit different.

> Question: Do the value constructors need to be capitalized?

Yes! They do. We'll talk more about that in a sec. But the constructors need to be capitalized, and the types themselves need to be capitalized. The reason we haven't seen any lowercase types yet is that Types, like `Sorting`, `String`, etc. need to be capitalized in order to indicate that they are in fact **Types**.

So, when we say that in our example that `Sorting` takes 3 different types, we're saying that these are completely new values which haven't existed anywhere else in the program. So it's _only_ true that `Ascending == Ascending`, while all other comparisons are false.

Likewise, if we try to compare these types to anything else, ie. `Descending == Number`--it will always be `false`. Specifically its a compile-time ERROR, or Type mis-match.

What common between the members of a **Union Type** is that they are themselves of the same type. So that `Ascending`, `Descending` and `Randomized` are all of the **Type** `Sorting`.

Here's another Union Type which shows how `Bool` is actually defined within Elm.

```elm
type Bool
    = True
    | False
```

We tend to put the union types on separate lines just so that file diffs are easier to read.

This also explains a bit of why the `True` and `False` types in Elm are capitalized, whereas they are lowercase in JavaScript. This is because in Elm, they are in fact _types_, where in JS they are _keywords_ and have a little bit different meaning to the language. In Elm, its just a "plain old Union Type, like anything else".

Further, the members of a Union Type are **CONSTANTS**--they can be used throughout the program, where are the Type signature is a type and has no meaning outside of the type annotation

[Image showing Bool <- Type; True and False <- constants]

Same with the first example, you can pass 'Randomized', 'Ascending' and 'Descending' to functions, put them in expressions, etc--'Sorting' only goes in Type expressions.

### Using Case Statements with Union Types

Since Union Types only accept one of their specified values as being valid, there's no need for a "default" branch if you want to use a case expression to check for one of those values.

```elm
case currentSorting of
    Ascending ->
        -- sort ascending here
    Descending ->
        -- sort descending here
    Randomized ->
        -- sort randomized here
```

No need for a `_` default branch since the type `Sorting` wouldn't know what to do with it anyway. It _can only_ be one of those three constants. You may have *no more than*--and--*no fewer than* the exact values of the union type as annotated.

If you were to leave off one of the possible values, Elm will let you know that that case has not been handled. Another great way that Elm lets us avoid bugs by not missing possible states within our programs.

This is especially good when we might want to introduce a new type within the union type, and Elm immediately throws errors wherever that type is used, rather than having to hunt down all of the places manually.

#### Defining argument types for union type values

It might be a good idea to indicate what type of value that a member of a union type might accept.

```elm
type Sorting
    = Ascending String
    | Descending String
    | Randomized
```

Essentially means means we are _parameterizing_ the type constructors. We're saying "Not only do I need to have Ascending, but Ascending on which column" (column being the String passed to the Ascending/Descending branch). Randomized wouldn't need to be followed by an argument since its own behavior is constant.

Doing this means that `Ascending` and `Descending` are now _functions_, rather than _constants_ as they were before parameterizing them. `Randomized` is still a constant.

So the type of this function types is now `String -> Sorting`.

Here's what you can do to extract the values once they're defined:

```elm
case currentSorting of
    Ascending colName ->
        -- sort ascending here
    Descending colName ->
        -- sort descending here
    Randomized ->
        -- sort randomized here
```

This is similar to the destructuring that we talked about when we covered Tuples in part one. Now, within the scope of the `Ascending` branch we can refer to the `colName` argument and have access to it _only_ within that scope.

The way that Elm's parser knows that we're talking about a specific value **is** that lowercase letter--`colName`--whereas if were to pass another type, ie. "`Ascending True`", that wouldn't destructure the argument and we would not have access to it. This is used when defining the constructor in the definition of the union type.

> Question: If I wanted for example _two_ cases that were ascending, but I wanted an exact column name instead of any String--can I do that?

Yeah! You totally can, for example:

```elm
case currentSorting of
    Ascending "first" ->
        -- sort just the first column here
    Ascending colName ->
        -- sort any other column here
    ...
```

Now we need to be careful because the compiler will force us to cover all possible cases, therefore you'll need either a default branch or be sure to cover the new possibility in an explicit way.

## A real-world example of Union Types

> Video: Messages and audience questions

Let's cover a few more topics, namely:

- Type Aliases
- Messages
- Records

```elm
type alias Msg =
    { operation : String
    , data : Int
    }

-- an example of a record whose types match

{ operation = "DELETE_BY_ID"
, data = 3
}

-- That's fine. What if, however--we wanted to perform another type of operation?

{ operation = "SET_QUERY"
, data = "tutorial"
}

-- This is when Union Types will come in handy.
```

In this example from the codebase, the user is going to type in a _query_ to be sent to the GitHub API. These are both a type of `operation`, but one is clearly dealing with `Int` values and the other with `String` values.

One solution might be to split that models, ie. `int-data` and `string-data` and make one or the other some nonsense value depending on which branch we wanted to execute.

> Or, we could do this!

```elm
type Msg
    = SetQuery String
    | DeleteById Int
```

We can present our `Msg` _as_ a **Union Type**. Remember that Union types can take any type at their values. The cool part about this is that:

1. Each individual message is only holding the data that it needs
2. This scales for as many different `Msg` types as we want

Here's some logic which utilizes the Union typing,

```elm
case msg of
    SetQuery query ->
        -- set query in the model here
    DeleteById id ->
        -- delete the result with this id here
```

We'll do exactly the same operation which showed up in Part 3, where we'll take `DeleteById` and filter the model accordingly.

This is the preferred way to handle **messages**.





[contentious thing]: https://github.com/iteles/Javascript-the-Good-Parts-notes#appendix-b---the-bad-parts
