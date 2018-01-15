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

* * Type mismatches errors can be hard to debug if they aren't precise enough. Extracting a small piece and annotating it can help you get more targeted error messages.



