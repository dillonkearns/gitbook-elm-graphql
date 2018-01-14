# Intro to [`elm-graphql`](/package.elm-lang.org/packages/dillonkearns/graphqelm/latest)

`elm-graphql` is an Elm package and accompanying command-line code generator that creates type-safe Elm code for your GraphQL endpoint. You don't write any decoders for your API with `elm-graphql`, instead you select which fields you would like to select, similar to a standard GraphQL query, but in Elm. For example, this GraphQL query

```graphql
query {
  human(id: "1001") {
    name
  }
}
```

would look like this in `elm-graphql`

```elm

import Graphqelm.Operation exposing (RootQuery)
import Graphqelm.SelectionSet exposing (SelectionSet, with)
import Swapi.Object
import Swapi.Object.Human as Human
import Swapi.Query as Query


type alias Response =
    { vader : Maybe Human
    }


query : SelectionSet Response RootQuery
query =
    Query.selection Response
        |> with (Query.human { id = Swapi.Scalar.Id "1001" } human)


type alias Human =
    { name : String
    }


human : SelectionSet Human Swapi.Object.Human
human =
    Human.selection Human
        |> with Human.name
```

 After installing the command line tool and Elm package, running `elm-graphql` just looks like

```bash
graphqelm https://graphqelm.herokuapp.com --base Swapi --output examples/src
```



