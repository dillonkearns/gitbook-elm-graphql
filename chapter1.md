# Selection Sets

At its simplest, a basic query is just a selection set

```graphql
query {
  myUsername
}
```

The `{ myUsername }` part is called a selection set. Since the `myUsername` field is just a simple String, it ends there. But if it were an object, you would need a selection set to say which fields you wanted to select from that object:

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
    Query.selection identity
        |> with (Query.viewer viewerSelection)


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
SelectionSet \(Maybe String\) RootQuery
```

Hint: Your type annotation uses type variable \`notSureYet\` which means any type of value

can flow through. Your code is saying it CANNOT be anything though! Maybe change

your type annotation to be more specific? Maybe the code has a problem?

