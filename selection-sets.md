# Selection Sets

### Plain GraphQL Selection Sets

_Definition:_ **Selection sets** are how you describe a set of **fields** to pull off of an **object**, **interface**, or **union** in GraphQL. We'll just be using **objects** in this chapter.

The Github API allows you to query `viewer` to get details for the currently logged in user.

```graphql
query {
  viewer {
    name
  }
}
```

Here, `{ name }` is a **selection set** that tells GraphQL to give you the `name` **field** from the `viewer`.

In fact, when you specify a top-level **query** in GraphQL you're simply specifying a **selection set**. You may have guessed based on the definition above then that a **query** is a GraphQL **object **since we are using a **selection set** on it, and you'd be right! A **mutation** is also defined as an **object.**

**Selection sets** are often nested. The top-level **selection set** in our example refers to another **object** \(`viewer`\) so we need a nested **selection set** to tell it we want the `viewer`'s name. Since the `name` field is a simple String, the nesting stops there.

### Elm `SelectionSet`s

`elm-graphql` will generate a module for each **object** in your server's schema \(including the **mutation** and **query** **objects**\). Each **object's** module has a `selection` function which starts a pipeline for building up a `SelectionSet`. This pipeline pattern is based on the [`Json.Decode.Pipeline` pattern](https://github.com/NoRedInk/elm-decode-pipeline).

Retrieving a simple integer value from a top-level **query** looks something like this:

```haskell
type alias Response = 
    { answer : Int }

query : SelectionSet Response RootQuery
query =
    HitchHiker.Query.selection Response
        |> with HitchHiker.Query.answerToLifeTheUniverseAndEverything
```

As with the `Json.Decode.Pipeline` library, it is common to use a constructor function \(like we do with `Response` in this example\) to start the pipeline. When we make a type alias for a record in Elm, we automatically get a function of the same name that takes each record attribute as an argument and builds up a record. So with `type alias Person = { first : String, last : String, age : Int }` we can build `Person` record with its constructor function like this: `Person "Arthur" "Dent" 42` \(returns `{ first = "Arthur", last = "Dent", age = 42 }`\).

We could have given any function that takes an `Int` as the argument in place of the `Response` constructor.

```haskell
query : SelectionSet Int RootQuery
query =
    HitchHiker.Query.selection identity
        |> with HitchHiker.Query.answerToLifeUniverseAndEverything
```

The identity function in Elm simply takes a thing and passes it right back. It's essentially a one-liner, here's the code:

```haskell
identity : a -> a
identity x =
  x
```

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

Let's walk through the process of building up the same **selection set** from our earlier example in Elm.

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

`Debug.crash` may cause our program to fail at run-time, but it is actually a neat trick for getting our program to succeed at compile-time. It essentially tells the compiler to not check the pieces that we're still working on. That let's us check that what we've completed so far compiles and take smaller steps.

We're also omitting the type annotations for now. Once we finish building our **selection sets**, the compiler will actually be able to infer the types for us so there's not any reason to worry about it yet.

We're almost there, we just need to fill in our `Debug.crash "TODO"`. The definition of the `Github.Query.viewer` function tells us that it needs an argument of type `SelectionSet decodesTo Github.Object.User` . This makes sense, just as with our plain GraphQL syntax above we need a **selection set** that tells our server which fields we want to get back from the `viewer`. We really just want to give it a **selection set** that looks like `{ name }` . But `elm-graphql` can't just let you pass any field anywhere, it makes sure your query is valid by ensuring that your **selection set** has **fields** from the correct **object**. That's what `SelectionSet decodesTo Github.Object.User` means. So we know we'll need to start with

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

We can use our editor's auto-complete functionality, or just manually inspect the code in the module `Github.Object.User` to see which **fields** are available.

![](/assets/nameField.png)That's just what we're looking for! Since we're looking for a `Field` to add to a `SelectionSet decodesTo Github.Object.User`, we can actually use any `Field` defined in the `Github.Object.User` module.

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


