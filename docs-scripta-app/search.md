# Search

## Search by Abstract

The abstract of a document summarizes information
about a document, e.g, its title, author and tags.
Thus a search by abstract with query "einstein" might match
"einstein" in the title, in the author field, or in a document tag.

Queries for documents using [jxxcarlson/elm-text-search](https://package.elm-lang.org/packages/jxxcarlson/elm-text-search/latest/)
look like the following:

- "foo": retrieve documents whose abstract digest contains "foo"


- "foo bar": retrieve documents whose abstract digest contains "foo"
and "bar"


- "foo -bar": retrieve documents whose abstract digest contains "foo"
  but not "bar"


- "foo | bar": retrieve documents whose abstract digest contains "foo"
    or "bar"


- "'foo bar'": *Note the single quotes!* — retrieve documents whose abstract digest contains the
phrase "foo bar".  This search excludes documents containing
"bar foo", "foo baz bar", etc.

To be technical, any query in disjunctive normal form is
is accepted.

In addition, a query can contain the fragment ":r", 
which signifies a random sample (max 20) of the documents
which match the search query.  Thus the query "atom:r" returns
a random sample of documents which match "atom", while
the query ":r" returns a random sample of documents
matching the ambient conditions, e.g., begin public or
belong to the current user.  For more complex queries,
the fragment ":r" can appear by itself.


## Document Abstracts

A document abstract has type

```
type alias Abstract =
    { title : String
    , author : String
    , abstract : String
    , tags : String
    , digest : String }
```

The `digest` is computed by

```
toString : Abstract -> String
toString a =
    [ a.title, a.author, a.tags ] |> String.join "; "

```

Abstracts are held in the field 
`abstractDict : Dict DocId Abstract` of the 
backend model.  This dictionary is updated 
every `Config.backendTickSeconds` by the 
update function of the backend model:

```
Tick newTime ->
    -- Do regular tasks
    ( { model | currentTime = newTime }
        |> updateAbstracts
        |> Backend.Update.updateDocumentTags
    , Command.none
    )
```

## Initiating a Search by Abstract



User document searches are initiated by the `Search` clause of
`Frontend.update`.  This clause invokes `Frontend.Search.search`
which send a message of type

```text
SearchForDocuments DocumentHandling SearchMode QueryString
```

to the back end, where `QueryString` is an alias for `String`
and where

```text
type SearchMode
    = UserSearch Username
    | PublicSearch
```

This message is handled by



```text
-- module Backend.Search
forDocuments : 
  BackendModel 
  -> ClientId 
  -> DocumentHandling 
  -> SearchMode
  -> Types.QueryString 
  -> ( BackendModel, Command BackendOnly ToFrontend backendMsg )

```

All searches that can be routed through this function
should be. 


## Search by Tag


((Under Construction))