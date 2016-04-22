# dropbox-js
Javascript implementation of Dropbox APIv2 using Dropbox REST api.

This implementation directly follows the official Dropbox API documentation found at: https://www.dropbox.com/developers/documentation/http/documentation. <br />
It is simply a wrapper around XMLHttpRequest that adjusts the request and response handling to account for variances in the Dropbox API (like different API endpoint URLs, expected requeset headers, etc.) and exposes a consistent javascript API to make the requests.

**Though the entire Dropbox API is accessible, only a subset has been exercised through this tool. Please report any bugs and I will do what I can to resolve them quickly.**

## Usage
```
dropbox(apiFunction, apiArguments, handlers);

// ex:
dropbox('files/list_folder', {path: '/some/path'}, function(result){ console.log(result.entries); });
// or
dropbox('files/list_folder', {path: '/some/path'}, {
  onComplete: function(result){ console.log(result.entries); },
  onDownloadProgress: function(result){ console.log(result.entries); }
);

```

The values for `apiFunction` are exactly as documented in the official Dropbox API documentation. Likewise, the `apiArguments` accepted are the same as documented on that page. 

`handlers` can either be a single function (assumed to be `onComplete` equivalent), or an object of functions. Currently supported: `onComplete`, `onError`, `onDownloadProgress` and `onUploadProgress`(for browsers which support it)<br />
`onComplete` receives the JSON API response, the response payload, and a reference to the XMLHttpRequest object.<br />
`onError` recieves a reference to the XMLHttpRequest object.<br />
`onDownloadProgress` and `onUploadProgress` are directly attached to the XMLHttpRequest object. see: https://developer.mozilla.org/en-US/docs/Web/Events/progress

## Authentication
```
dropbox.authenticate('client_id', handlers);
//or
dropbox.authenticate( { client_id: 'client_id', redirect_uri: 'http....' }, function(){ /* successfully authenticated */ } );
```

Currently, only `token` authentication is supported by this library. The authentication code currently assumes it is executing in a browser and uses the `state` parameter to implement the CSRF token. I intend to expose `state` to the `dropbox.authenticate` method for use by the consumer eventually.

`handlers` can either be a single function (assumed to be `onComplete` equivalent), or an object of functions. Currently suppoorted: `onComplete`, `onError`.<br />
`onComplete` receives no parameters.<br />
`onError` receives an object with the deserialized URL hash params which should include `error` and `error_description` from Dropbox.

## Global Error Handling
Rather than handling errors at every API call, you can register a global error handler.
```
dropbox.setGlobalErrorHandler(handler);
```

The global error handler, if specified, is called after the call-specific error handler is called. It is passed the same parameter(s) as the call-specific error handler, plus the return value of the call-specific error handler. For example, you can shortcut global error handling for a specific API call like this:
```
dropbox.setGlobalErrorHandler( function(http, alreadyHandled){ if(alreadyHandled){ return; } } );

dropbox('files/list_folder', { path: '' }, { onError: function(){ return true; }});
```

## Token Storage
By default, the CSRF token and the user_access token returned by Dropbox after successful authentication are stored in localStorage. You can override this by injecting a custom token store. A token store is simply a function that takes one or two arguments (i.e. `key` and `value`) and gets or sets the value for the key as appropriate. If called with only one parameter, the value should be returned and when called with two parameters, the value should be set.
```
dropbox.setTokenStore( customStore )

// ex:
var localStorageTokenStore = function(key,val){
  return ( arguments.length > 1 ) ? (localStorage[key] = val) : localStorage[key];
}
dropbox.setTokenStore( localStorageTokenStore );
```
