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

When defining `SelectionSet`s in Elm, it is sometimes easier to start from the leaves and work your way up. This can provide more precise error messages when you're figuring out how to define your records. Let's try building the above example with `elm-graphql`.







* * Type mismatches errors can be hard to debug if they aren't precise enough. Extracting a small piece and annotating it can help you get more targeted error messages.



