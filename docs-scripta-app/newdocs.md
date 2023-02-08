# Making a New Document

A new document may stand by itself or be intended as a
member of an existing smart folder or collection of 
documents.  The `Frontend.Update.newDocument` must
take into account these eventualities:

- Suppose that the current master document is a smart folder.
  Then creation of a new document should attach that document
  to the smart folder.  See `refreshSmartFolderCmd`.

- Suppose that the current master document is a collection
  and that one of its members is the current document.
  Then creation of a new document should add it to the collection,
  and it should be positioned immediately after the
  current document. See `updateCollectionCmd`.

In both cases, the new document should be the selected
document in the index for the folder or collection (this
is not currently the case.)