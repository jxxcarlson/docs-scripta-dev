# Lamdera

As a guide to what follows, we describe briefly the 
architecture of a Lamdera app.  The basic idea
is that there is a frontend and a backend, the two 
communicating via a message protocol over websockets.
Thus, to fetch a document from the backend, the frontend
may issue the command

```
Lamdera.sendToBackend (FetchDocumentById SimpleHandling doc.id)
```

The backend would then retrieve the document using the id,
and if found, send it to the frontend via

```
Lamdera.sendToFrontend clientId (ReceivedDocument SimpledHandling document)
```