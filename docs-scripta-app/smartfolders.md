# Smart Folders

A smart folder is a way of automatically producing a collection
document, where the documents in the collection are those
(belonging to the user) with a specified tag.  Such a 
collection is created at the backend by the function 
`SmartFolder.makeCollectionDocumentCmd`.   This function
is invoked indirectly from the frontend by the function
`SmartFolder.makeCollectionDocument`.



The format of a smart folder document is

```text
| title
Notes (Foo)

[tags :folder, gettag:foo]
```

The resulting collection document will be populated by all of
the user's documents tagged with "foo".
The old format is for the time being still
operational:

```text
| title
Notes (Foo)

[tags :folder]

| type folder get:foo ; 
```

## Creating a Smart Folder

A smart folder is created by the function
`SmartFolder.create`.  It creates the document
and sends to to the backend via the message
`CreateDocument`, which invokes the function
`Backend.Document.create`


## Sort options

The default sort option is to sort titles 
alphabetically (A to Z).  The sort option
may be changed by specifying a tag:

- `sort:created-oldest-first`
- `sort:created-newest-first`
- `sort:modified-oldest-first`
- `sort:modified-newest-first`
- `sort:a-to-z`
- `sort:z-to-a`


## Rendering a smart folder

When the user click on a smart folder, the normal 
chaing of function calls is activated to make it the 
current document.  (See the section on [opening a document](/openDoc/))

