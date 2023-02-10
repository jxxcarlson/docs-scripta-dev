# Codemirror Integration

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
reads this value and uses it to scroll the given line into view and highlight it
(see `HtmlAttr.attribute "linenumber" ... ` below).

**TODO:**
  - The line in question needs not only to be highlighted
and brought into view, but also be centered vertically on the
page.
  - We need a more granular LR sync -- ideally on the word or
phrase level.
  - 
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