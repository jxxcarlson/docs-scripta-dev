# Codemirror Integration

## Custom Element



## Communication between Elm and Codemirror

## Codemirror side

On the Codemirror side, communication with Elm is handled by
function `AttributeChangedCallback(attr, oldVal, newVal)`.


### Elm side

The handler is `View.Editor.view`:


**Receive information from Codemirror**

- `E.htmlAttribute onSelectionChange`
- `E.htmlAttribute onTextChange`
- `E.htmlAttribute onCursorChange`

**Send information to Codemirror**

- `HtmlAttr.attribute "text" model.initialText`
- `HtmlAttr.attribute "editordata" (fixEditorData model.language model.editorData |> encodeEditorData)`
- `HtmlAttr.attribute "selection" (stringOfBool model.doSync`
- `HtmlAttr.attribute "editcommand" (OTCommand.toString model.editCommand.counter model.editCommand.command`


## Left to Right Sync

### Data flow

1. User presses ctl-S; A `KeyMsg keyMsg` is received and passed to `EditorSync.handleKeypress`
2. `handleKeypress` inspects the message.  If it is ctrl-S, `model.doSync` is toggled.  This
is a signal, via `HtmlAttr.attribute "selection" (stringOfBool model.doSync)` in `Editor.view`
to Codemirror send the text that has been selected in the edtor back to Elm .  Codemirror
does this via function `sendSelectedText` which creates the event "selected-text"
3. The "selected-text" event is detected by the editor which sends the Elm message `SelectedText str`
to the frontend.  That message calls `Frontend.EditorSync.firstSyncLR model str`
This function issues a command to bring ....


### Code

- `model.doSync`
- Message `StartSync`: toggle `model.doSync`
- Message `NextSync`

```elm
SelectedText str ->
    Frontend.EditorSync.firstSyncLR model str

SendSyncLR ->
    ( { model | syncRequestIndex = model.syncRequestIndex + 1 }, Effect.Command.none )

SyncLR ->
    Frontend.EditorSync.syncLR model

StartSync ->
    ( { model | doSync = not model.doSync }, Effect.Command.none )

NextSync ->
    Frontend.EditorSync.nextSyncLR model
```

## Sending information to Codemirror

The view function for a document is

```elml
viewDocument : 
   Scripta.API.DisplaySettings 
   -> Compiler.DifferentialParser.EditRecord 
   -> List (Element FrontendMsg)
viewDocument displaySettings editRecord =
    Scripta.API.render displaySettings editRecord 
      |> List.map (E.map Render)    
```

where in module `Types` we find the

```elm
-- Render.Msg
type MarkupMsg
    = SendMeta { begin : Int, end : Int, index : Int, id : String }
    | SendLineNumber Int
    | SelectId String
    | HighlightId String
    | GetPublicDocument Handling String
    | GetPublicDocumentFromAuthor Handling String String
    | GetDocumentWithSlug Handling String
    | ProposeSolution SolutionState
```


When a user clicks on rendered text, a message of type 
`Render (SendLineNumber Int)` is sent to the update function of Scripta.
The clause `Render msg_` of the update function calls
`Frontend.Update.render model msg_`.  In the case of 
`Render.Msg.SendLineNumber line` the handler is

```elm
  -- Frontend.Upate
  Render.Msg.SendLineNumber line ->
    ( { model
        | selectedId = ""
        , messages = [ { txt = "Line " ++ String.fromInt (line + 1), status = MSGreen } ]
        , editorLineNumber = String.fromInt line
      }
    , Command.none
    )
```

The important part is the line `editorLineNumber = String.fromInt line`.
This value is stored in the model.  The view function for the editor subsequently
reads this value and communicates it to Codemirror 
it to scroll the given line into view and highlight it.  
Here is the relevant part of `editor.js`:


(see `HtmlAttr.attribute "linenumber" ... ` below).

```javascript
case "linenumber":
     console.log("!!!@ IN linenumber !")
      // receive info from Elm (see Main.editor_)
      // scroll the editor to the given line
       var lineNumber = parseInt(newVal) + 2
       var loc =  editor.state.doc.line(lineNumber)
       console.log("Attr case lineNumber", loc)
       console.log("position", loc.from)
       editor.dispatch({selection: {anchor: parseInt(loc.from)}})
       editor.scrollPosIntoView(loc.from + 400)
    break
```

**TODO:**

  - The `+ 400` part of the code ensures that the source text is scrolled up 
somewhat from the bottom margin of the editor.  We need to make this conditional
on the line number, so that if the line number is too small, the text is not
scrolled up too far.

  - We need a more granular LR sync -- ideally on the word or
phrase level.

```elm
-- View.Editor
view : FrontendModel -> Element FrontendMsg
view model =
    Element.Keyed.el
        [ -- RECEIVE INFORMATION FROM CODEMIRROR
          E.htmlAttribute onSelectionChange 
        , E.htmlAttribute onTextChange 
        , E.htmlAttribute onCursorChange 
        , htmlId "editor-here"
        ...
        ]
        ( stringOfBool model.showEditor
        , E.html
            (Html.node "codemirror-editor"
                [ -- SEND INFORMATION TO CODEMIRROR
                , HtmlAttr.attribute "linenumber" (shiftLineNumber model.language model.editorLineNumber)
                , HtmlAttr.attribute "selection" (stringOfBool model.doSync)
                , HtmlAttr.attribute "editcommand" (OTCommand.toString model.editCommand.counter model.editCommand.command)
                ]
                []
            )
        )
```