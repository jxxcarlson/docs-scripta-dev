# PDF Server

Scripta exports a document to PDF by converting it to standard 
LaTeX, then sending via a POST request to `pdfServe.app`.  
The server responds with a link that will retrieve the 
document via a GET request.

## Server code

The server is written in Haskell (175 LOC), with the code residing
at [github.com/jxxcarlson/pdfServer2](https://github.com/jxxcarlson/pdfServer2).
The code is open source.



## Server deployment

The server is deployed to Digital Ocean at 

## Running and testing the server locally

