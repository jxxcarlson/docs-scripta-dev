# Loading data into a document

The now-official way to load data into a document is to use 
a `setup` block:

```
|| setup
named-data: 
  hubble https://dataserv.app/hubble.csv
  weather https://dataserv.app/weather.csv

```

Loading named data into a document requires changes to both 
the Scripta app and the Scripta compiler.

## Scripta app

First we define new types for a field `setupDict: SetupDict` which is
added to the frontend model.  The keys are strings and values a
are one of `String String` or `StringList (List String)`:

```elm
type Val
    = String String
    | StringList (List String)


type alias SetupDict =
    Dict String Val

```

One creates a `SetupDict` by parsing a list of strings using

```elm
SetupDocument.makeDict : List String -> Dict String Types.Val
```
In `CurentDocument.setDocumentAsCurrent_`, apply this function
to `doc.content`, update the model with it, and issue the command
`Frontend.Cmd.loadNamedData` with the resulting `SetupDict`.
That command will send a batch of requests, one for each named
data source, that request data from the given URLs using 
`Frontend.Cmd.getData label url` where the expectation is

```elm
Effect.Http.expectString (GotData label)
```
The `GotData` message is handled by `Frontend.update` function
in clause `GotData label data` which updates the model with
`data = Dict.insert label data model.data`.

## Scripta Compiler 

Here is an outline of how the Scripta compiler works to render
a chart with a remote data source.  First, when
the document is made current, the `setup` block is parsed and
the resulting `SetupDict` is added to the frontend model.  
Second, when the document is rendered, (by `View.Document.view`),
the `displaySettings` are initialized with the `data` field
set to `model.data`.  The view function calls
`viewDocument width_ displaySettings model.editRecord`,
where `model.editRecord` contains the parsed source text of 
the document and `displaySettings` contains the dictionary for
named data sources as its `data` field.   

```elm
renderSettings =
            ScriptaV2.Settings.renderSettingsFromDisplaySettings displaySettings
```

in `DifferentialCompiler.editRecordToCompilerOutput`.


When a V2 chart is rendered
by Render.ChartV2.render, we first extract the name of the data
source by the call `Dict.get "source"` on the `properties` field of the
chart block.  If this call is successful, yielding `name`, we 
call `Dict.get name settings.data` where `settings.data` is 
XXX derived from model.data (see below).  If this call yields a valid set of data `data`, we
make the call `chart kind properties data` where `kind` describes
the kind of chart to make, e.g., `line`, `bar`, `scatter`, etc.