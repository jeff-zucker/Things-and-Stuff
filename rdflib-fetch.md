# Using authenticated and alternate fetches in rdflib

By default, rdflib's fetcher uses cross-fetch.fetch() to preform reads and writes.  This is a plain fetch similar to wget or curl that does not carry any authentication information.  In order to use rdflib with private Solid data that requires authentication, the user or app needs to pass a different fetch method to rdflib.  This approach is also useful if you have created your own fetch, for example against Dropbox or a database.                                                                       

## The solidFetch variable                                                

To to use an authenticated or alternate fetch with rdflib

1. load the authentication library
2. use it to login to your identity provider
3. set the global.solidFetch (or window.solidFetch in a browser) equal to the auth library's fetch method
4. load and use rdflib, all fetches will be authenticated

**Important Note :** prior to rdflib version 2.2.9 this variable was named solidFetcher so use that with older rdflibs.  Going forward, please use solidFetch.

### A CLI (out-of-browser) Example

```javascript                                                                   
async function test() {
    const auth = new (require("solid-node-client").SolidNodeClient)();
    await auth.login( your-credentials );                             
    global.solidFetch = auth.fetch;                                   
    const $rdf = global.$rdf = require('rdflib');                     
    const kb = $rdf.graph(); 
    const fetcher = $rdf.fetcher(kb);
    await fetcher.load( some-private-url );  
}
```               

### A Browser Example

This is a fully functinal script, just change the IDP and privateResource addresses.

```html
<!DOCTYPE html><html><head><meta charset="UTF-8" />                             
    <script src="https://cdn.jsdelivr.net/npm/@inrupt/solid-client-authn-browse\
r@1.11.2/dist/solid-client-authn.bundle.js"></script>                           
    <script src="https://cdn.jsdelivr.net/npm/rdflib@2.2.6/dist/rdflib.min.js">\
</script>                                                                       
</head><body>                                                                   
    <button id="login">go</button>                                              
</body>                                                                         
<script>                                                                        
    const idp = "https://solidcommunity.net";                                   
    const privateResource = "https://jeff-zucker.solidcommunity.net/private/";  
    const auth = solidClientAuthentication;                                     
    async function main(session){                                               
        const kb = $rdf.graph();                                                
        window.solidFetcher = session.fetch;                                    
        const fetcher = $rdf.fetcher(kb)                                        
        try {                                                                   
          await fetcher.load(privateResource);                                  
          alert("Private resource successfully loaded");                        
        }                                                                       
        catch(e) { alert(e) }                                                   
    }                                                                           
    document.getElementById('login').onclick = ()=> {                           
        auth.login({                                                            
            oidcIssuer: idp,                                                    
            redirectUrl: window.location.href,                                  
            clientName: "rdflib test"                                           
        });                                                                     
    }                                                                           
    async function handleRedirectAfterLogin() {                                 
        await auth.handleIncomingRedirect();                                    
        session = auth.getDefaultSession();                                     
        if (session.info.isLoggedIn)  main(session);                            
    }                                                                           
    handleRedirectAfterLogin();                                                 
</script></html>                                                                
```


##  Custom fetchers

Another way to specify an authenticated or alternate fetch is to do it when you create the fetcher.  This method follows the same steps as the first method with two exceptions : omit the line *global.solidFetch = auth.fetch* and instead of the line *const fetcher = $rdf.fetcher(kb)* use :
```javascript
const fetcher = $rdf.fetcher(kb,{fetch:auth.fetch.bind(auth)});
```

## Currently available libraries with Solid fetch methods

* [Inrupt's Solid-Client-Authn-Browser](https://github.com/inrupt/solid-client-js) for use in browsers
* [Inrupt's Solid-Client-Authn-Node](https://github.com/inrupt/solid-client-js) for use in node when you don't need full access to the local file system
* [Solid-Node-Client](https://github.com/solid/solid-node-client) for use in node when you need full access to the local file system
