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

When defining `SelectionSet`s in Elm, you get more precise type error messages when you break off small pieces into constants as you go. We'll take the smallest possible steps to make sure our code is compiling early and often. That way we'll get better and more frequent feedback from the compiler so we don't have to dig deep to find and fix our mistakes. The Elm compiler usually can't give us very good feedback if we take huge steps with lots of pieces that don't fit together so we'll optimize for compiler feedback and keep our steps and our functions/constants small.

```haskell
query : SelectionSet notSureYet RootQuery
query =
    Github.Query.selection identity
        |> with (Github.Query.viewer viewerSelection)

viewerSelection = Debug.crash "TODO"
```

`Debug.crash` may cause our program to fail at run-time, but it is actually a neat trick for getting our program to compile. It essentially tells the compiler to not check the pieces that we're still working on. That let's us check that what we've completed so far compiles and let's us get around to another piece later.

Since `notSureYet` is lowercase, it's a type variable, which just means it's a placeholder for any type. Once we finish building our **selection set**, the compiler will actually be able to infer the type for us so there's not any reason to worry about it yet. We could also omit the  type annotation like we did for the `viewerSelection` constant.

The `Debug.crash` is handy here because Elm treats its type in a special way. It's like a wild card, it has whatever type it needs to. For example, if we wrote `1 + viewerSelection`, it would compile, and the compiler would suggest that we annotate `viewerSelection` as a number.

The definition of the `Github.Query.viewer` function tells us that we need a `SelectionSet decodesTo Github.Object.User` . This makes sense, just as in our plain GraphQL syntax above, we need a **selection set** that tells our server which fields we want to get back from the User `viewer`. We really just want to give it a **selection set** that looks like `{ name }` . But `elm-graphql` can't just let you pass any field anywhere, it makes sure your query is valid by ensuring that your **selection set** has **fields** from the correct **object**. That's what `SelectionSet decodesTo Github.Object.User` means. We can use our editor's auto-complete functionality, or just manually inspect the code in the module `Github.Object.User` to see which **fields** are available.

![](/assets/Field_autocompletion.png)

That's just what we're looking for! But it looks like this is a `Field (Maybe String) Github.Object.User`, not a `SelectionSet`. So this gives us just a **field**, not a selection set. It's the equivalent of `name` instead of `{ name }`. So how do we build up a `SelectionSet` from a `Field`?

```haskell
type alias User =
    { name : Maybe String }


viewerSelection : SelectionSet Viewer Github.Object.User
viewerSelection =
    Github.Object.User.selection User
        |> with Github.Object.User.name
```

If you're using the Atom editor and the Elmjutsu plugin, typing `Query.viewer` will autocomplete with the types of the argument, which are `(SelectionSet selection Github.Object.User)`. This means that the `Query.viewer` function needs a `SelectionSet` of fields from the `Github.Object.User` module. This makes sense because the viewer is an object, and objects need a selection of fields.

```graphql
query {
  viewer { # we're using Query.viewer, but we need to fill in the selection
    name   # name is a leaf, so once we add this to our selection we are done
  }
}
```

```haskell
# this 
query : SelectionSet notSureYet RootQuery
query =
    Github.Query.selection identity
        |> with (Github.Query.viewer viewerSelection)


viewerSelection : SelectionSet (Maybe String) Github.Object.User
viewerSelection =
    Github.Object.User.selection identity
        |> with Github.Object.User.name
```

The definition of \`query\` does not match its type annotation.

query : SelectionSet a RootQuery

query =

&gt;    Query.selection identity

&gt;        \|&gt; with \(Query.viewer viewerSelection\)

The type annotation for \`query\` says it is a:

```
SelectionSet notSureYet RootQuery
```

But the definition \(shown above\) is a:

```
SelectionSet (Maybe String) RootQuery
```

Hint: Your type annotation uses type variable \`notSureYet\` which means any type of value

can flow through. Your code is saying it CANNOT be anything though! Maybe change

your type annotation to be more specific? Maybe the code has a problem?

