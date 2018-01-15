# Selection Sets

### Plain GraphQL Selection Sets

**Selection sets** are how to you describe a set of **fields** to pull off of an **object** in GraphQL. The Github API allows you to query `viewer` to get details for the currently logged in user.

```graphql
query {
  viewer {
    name
  }
}
```

Here, `{ name }` is just a **selection set** that tells GraphQL to give you the `name` **field** for the `viewer`.

In fact, any top-level query is itself nothing more than a **selection set** in GraphQL \(in this case, the **selection set** itself refers to an **object** \(`viewer`\) so we need a nested **selection set** to tell it we just want the `viewer`'s name\). Since the `name` field is just a simple String, it ends there.

### Elm `SelectionSets`

Let's walk through the process of building up the same **selection set** as above in Elm.

When defining `SelectionSet`s in Elm, you get more precise type error messages when you break off small pieces into constants as you go.

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

