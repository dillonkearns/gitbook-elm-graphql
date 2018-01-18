# Selection Sets

### Plain GraphQL Selection Sets

_Definition:_ **Selection sets** are how you describe a set of **fields** to pull off of an **object**, **interface**, or **union** in GraphQL. We'll only see examples of **selection sets** on **objects** in this chapter.

The Github API allows you to query `viewer` to get details for the currently logged in user.

```graphql
query {
  viewer {
    name
  }
}
```

Here, `{ name }` is a **selection set** that tells GraphQL to give you the `name` **field** from `viewer`.

In fact, when you specify a top-level **query** in GraphQL you're simply specifying a **selection set**. You may have guessed based on the **selection set** definition then that a **query** is a GraphQL **object**,** **and you'd be right! A **mutation** is also defined as an **object** in GraphQL.

**Selection sets** are often nested. The top-level **selection set** in our example includes an **object**,`viewer` so we need a nested **selection set** to tell it which **fields** we want from `viewer` \(here we only wanted `name`\). Since the `name` **field** is a simple String, the nesting stops there.

### Elm `SelectionSet`s

`elm-graphql` will generate a module for each **object** in your server's schema \(including the **mutation** and **query** **objects**\). Each **object's** module has a `selection` function which starts a pipeline for building up a `SelectionSet`. This pipeline notation is based on the [`Json.Decode.Pipeline` pattern](https://github.com/NoRedInk/elm-decode-pipeline).

Retrieving a simple integer value from a top-level **query** looks something like this:

```haskell
type alias Response = 
    { answer : Int }

query : SelectionSet Response RootQuery
query =
    HitchHiker.Query.selection Response
        |> with HitchHiker.Query.answerToLifeTheUniverseAndEverything
```

As with the `Json.Decode.Pipeline` library, it is common to use a constructor function \(like `Response` in this example\) to start the pipeline.

> #### Constructor Functions
>
> If you're not familiar with constructor functions in Elm, they come for free when we create a type alias for a record. It is just a function with the same name as the type alias that takes each record attribute as an argument and builds up a record. So with `type alias Person = { first : String, last : String, age : Int }` we can build a `Person` record with its constructor function like this: `Person "Arthur" "Dent" 42` \(returns `{ first = "Arthur", last = "Dent", age = 42 }`\).

We don't have to use constructor functions, we could have used any function that takes a single `Int`  argument in place of the `Response` constructor function.

```haskell
identity : a -> a
identity x =
    x

query : SelectionSet Int RootQuery
query =
    HitchHiker.Query.selection identity
        |> with HitchHiker.Query.answerToLifeUniverseAndEverything
```

Here instead of wrapping the `answerToLifeUniverseAndEverything` value in a record, we're simply passing the raw `Int` through using our `identity` function. In fact, we didn't need to define `identity`  at all because it is a built-in function in the Elm language! You can use it anywhere in any Elm module.

Using the `identity` function is convenient when we want to avoid nesting our data any further. Using constructor functions is handy because it's easy to add on more fields incrementally.

```haskell
type alias Response = 
    { answer : Int, name : String }

query : SelectionSet Response RootQuery
query =
    HitchHiker.Query.selection Response
        |> with HitchHiker.Query.answerToLifeUniverseAndEverything
        |> with HitchHiker.Query.author
```

You'll develop a sense of when to use constructors versus other functions with practice.

### Building Up a `SelectionSet` in Elm

Let's walk through the process of building up our nested **selection set** from our earlier example within Elm.

```graphql
query {
  viewer {
    name
  }
}
```

When defining `SelectionSet`s in Elm, you get more precise type error messages when you break off small pieces into constants as you go. We'll take the smallest possible steps to make sure our code is compiling early and often. That way we'll get better and more frequent feedback from the compiler so we don't have to dig deep to find and fix our mistakes.

```haskell
query =
    Github.Query.selection identity
        |> with (Github.Query.viewer viewerSelection)

viewerSelection = Debug.crash "TODO"
```

`Debug.crash` may cause our program to fail at run-time, but it is actually a neat trick for getting our program to succeed at compile-time. It essentially tells the compiler to not check a piece that we're still working on. If it compiles, we know that everything besides the `Debug.crash` statements is looking good.

We're also omitting the type annotations for now. Once we finish building our **selection sets**, the compiler will actually be able to infer the types for us so there's not any need to worry about that yet.

We're almost there, we just need to fill in our `Debug.crash "TODO"`. The definition of the `Github.Query.viewer` function tells us that it needs an argument of type `SelectionSet decodesTo Github.Object.User` . This makes sense, just as with our plain GraphQL syntax above, we need a **selection set** that tells our server which fields we want to get back from the `viewer`. We want to give it a **selection set** that looks like `{ name }` . To ensure that your `SelectionSet` for `viewer` only has **fields** for a User **object**, `elm-graphql`  adds `Github.Object.User` to the `SelectionSet`'s annotation. That's what `SelectionSet decodesTo Github.Object.User` means. So we know we'll need to start with `Github.Object.User.selection`

```haskell
query =
    Github.Query.selection identity
        |> with (Github.Query.viewer viewerSelection)


viewerSelection =
    Github.Object.User.selection identity
        |> with nameField


nameField =
    Debug.crash "TODO"
```

We can use our editor's auto-complete functionality, or just manually inspect the code in the module `Github.Object.User` to see which **fields** are available in `Github.Object.User`.

![](/assets/nameField.png)That's just what we're looking for! Since we're looking for a `Field fieldDecodesTo Github.Object.User`  to add to a `SelectionSet selectionDecodesTo Github.Object.User`, we can actually use any `Field` defined in the `Github.Object.User` module.

Now we're all done! Since this is a `Field (Maybe String) Github.Object.User`, we could fill in the types ourselves at this point. Or we can let the compiler infer the types all the way through for us:

```haskell
query : SelectionSet (Maybe String) RootQuery
query =
    Query.selection identity
        |> with (Query.viewer viewerSelection)


viewerSelection : SelectionSet (Maybe String) Github.Object.User
viewerSelection =
    Github.Object.User.selection identity
        |> with nameField


nameField : Graphqelm.Field.Field (Maybe String) Github.Object.User
nameField =
    Github.Object.User.name
```



