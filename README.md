# dropbox-apiv2
Javascript implementation of Dropbox APIv2 using Dropbox REST api

This implementation follows the API documentation found at: https://www.dropbox.com/developers/documentation/http/documentation

## Usage
```
dropbox(apiFunction, apiArguments, handlers);

// ex:
dropbox('files/list_folder', {path: '/some/path'}, function(result){ console.log(result.entries); });
```

The values for `apiFunction` are exactly as documented in the official Dropbox API documentation. Likewise, the `apiArguments` accepted are the same as documented on that page. These are really all just passed through to the XMLHttpRequest sent to Dropbox, but in a way that adjusts the request and response handling to account for variances in the Dropbox API (like different API endpoint urls, expected request headers, etc.)

`handlers` can either be a single function (assumed to be `onComplete` equivalent), or an object of functions. Currently suppoorted: `onComplete`, `onDownloadProgress` and `onUploadProgress`(for browsers which support it)

## Authentication
```
dropbox.authenticate('client_id');
//or
dropbox.authenticate( { client_id: 'client_id', redirect_uri: 'http....' } );
```

Currently, only `token` authentication is supported by this library. The authentication code currently assumes it is executing in a browser and uses the `state` parameter to implement the CSRF token. I intend to expose `state` to the `dropbox.authenticate` method for use by the consumer eventually.

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
