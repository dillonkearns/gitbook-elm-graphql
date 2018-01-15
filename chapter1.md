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
  viewer {
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

The `Debug.crash` is handy here because Elm treats its type in a special way. It's like a wild card, it has whatever type it needs to. For example, if we wrote `1 + viewerSelection`, it would compile, and the compiler would suggest that we annotate `viewerSelection` as a number.

If you're using the Atom editor and the Elmjutsu plugin, typing `Query.viewer` will autocomplete with the types of the argument, which are `(SelectionSet selection Github.Object.User)`. This means that the `Query.viewer` function needs a `SelectionSet` of fields from the `Github.Object.User` module.

