# Using authenticated and alternate fetches in rdflib

By default, rdflib's fetcher uses cross-fetch.fetch() to preform reads and writes.  This is a plain fetch similar to wget or curl that does not carry any authentication information.  In order to use rdflib with private Solid data that requires authentication, the user or app needs to pass a different fetch method to rdflib.  This approach is also useful if you have created your own fetch, for example against Dropbox or a database.                                                                       

There are two ways to set rdflib's fetch method : 1) setting a global variable will impact all rdflib's fetches 2) creating a custom fetcher will impact all rdflib fetches with that fetcher.

## The solidFetch variable                                                

Here's how to declare the fetch once and thereafter use authenticated fetches in rdflib :

```javascript                                                                   
// load the auth library and set solidFetch to its fetch method                                              
// NOTE : these steps must be done before loading rdflib
//
const auth = new (require("solid-node-client").SolidNodeClient)();              
global.solidFetch = auth.fetch;                                                  
                                                                                
// load rdflib and create a fetcher                                     
//                                                                              
const $rdf = global.$rdf = require('rdflib');                                   
const kb = $rdf.graph(); 
const fetcher = $rdf.fetcher(kb);

// login and use rdflib methods with an authenticated fetch
// NOTE : you don't need to explicitly refer to auth after the login
//
await auth.login( your-credentials );
await fetcher.load( some-private-url );  
```               

##  Custom fetchers

The other way to specify an authenticated or alternate fetch is to do it when you create the fetcher.  This method follows the same steps as the first method with two exceptions : omit the line *global.solidFetch = auth.fetch* and instead of the line *const fetcher = $rdf.fetcher(kb)* use :
```javascript
const fetcher = $rdf.fetcher(kb,{fetch:auth.fetch.bind(auth)});
```
