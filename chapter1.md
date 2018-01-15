# Selection Sets

At its simplest, a basic query is just a selection set

```graphql
query {
  myUsername
}
```

The `{}`s and the contents are called a selection set. Since the `myUsername` field is just a simple String, it ends there. But if it were an object, you would need a selection set to say which fields you wanted to select from that object:

```graphql
query {
  currentUser {
    name
  }
}
```

When defining `SelectionSet`s in Elm, you get more precise type error messages when you break off small pieces into constants as you go. Let's walk through the process of building up the `currentUser` example above with `elm-graphql`.

```haskell
query : SelectionSet notSureYet RootQuery
query =
    Query.selection identity
        |> with (Query.viewer viewerSelection)

viewerSelection = Debug.crash "TODO"
```

Since `notSureYet` is lowercase, it's a type variable, which just means it's a placeholder for any type. Once we finish building our selection set, the compiler will actually be able to infer the type for us so there's not any reason to worry about it yet. We could also omit the  type annotation like we did for the `viewerSelection` constant.

The `Debug.crash` is handy here because it is a special statement in Elm which will

