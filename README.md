# google-oauth-jwt

Google OAuth 2.0 authentication for server-to-server applications with Node.js.

This library generates [http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html](JWT) tokens to establish
identity to an API, without an end-user being involved, which is the ideal scenario for server-side communications.
It can be used as a foundation for Google APIs requiring access to user data (such as Google Drive, Google
Calendar, etc.) for which web-based callbacks and user authorization prompts are not appropriate.

Tokens are generated for a service account, which is created from the Google API console. Service accounts must also
be granted access to resources, using traditional assignation of permissions using the unique service account email
address.

The authentication process is implemented following the specifications found
[https://developers.google.com/accounts/docs/OAuth2ServiceAccount](here).

The package also integrates with [https://github.com/mikeal/request](request) to seamlessly query Google RESTful APIs,
which is optional. Integration with [https://github.com/mikeal/request](request) provides automatic requesting of
tokens, as well as token caching.

## Documentation

### Installation
```bash
npm install google-oauth-jwt
```

### Generating a key to sign the tokens

1. From the [https://code.google.com/apis/console/](Google API Console), create a
   [https://developers.google.com/console/help/#service_accounts](service account).

2. Download the generated P12 key.
   IMPORTANT: keep a copy of the key, Google keeps only the public key.

3. Convert the key to PEM, so we can use it from the Node crypto module.
   To do this, run the following in Terminal:
   ```bash
   openssl pkcs12 -in downloaded-key-file.p12 -out your-key-file.pem -nodes
   ```

   The password for the key is `notasecret`, as mentioned when you downloaded the key.

### Granting access to resources to be requested through an API

In order to query resources using the API, access must be granted to the service account. Each Google application that
has security settings must be configured individually. Access is granted by assigning permissions to the service
account, using its email address.

For example, in order to list files in Google Drive, folders and files must be shared with the service account email
address.

### Querying a RESTful Google API with request

In this example, we use a modified instance of [https://github.com/mikeal/request](request) to query the
Google Drive API. The modified request module handles the token automatically using a `jwt` setting passed to
the `request` function.

```javascript
// obtain a JWT-enabled version of request
var request = require('google-oauth-jwt').requestWithJWT();

request({
	url: 'https://www.googleapis.com/drive/v2/files',
	jwt: {
    // use the email address of the service account, as seen in the API console
		email: 'my-service-account@developer.gserviceaccount.com',
		// use the PEM file we generated from the downloaded key
		keyFile: 'my-service-account-key.pem',
		// specify the scopes you which to access - each application has different scopes
		scopes: ['https://www.googleapis.com/auth/drive.readonly']
	}
}, function (err, res, body) {

	console.log(JSON.parse(body));

});
```

Note that the `options` object includes a `jwt` object we use to configure the token generation. The token will
automatically be requested and inserted in the query string for this API call. It will also be cached and
reused for subsequent calls using the same service account and scopes.

### Requesting the token manually

If you wish to simply request the token for use with a Google API, use the `authenticate` method.

```javascript
var googleAuth = require('google-oauth-jwt');

googleAuth.authenticate({
  // use the email address of the service account, as seen in the API console
  email: 'my-service-account@developer.gserviceaccount.com',
  // use the PEM file we generated from the downloaded key
  keyFile: 'my-service-account-key.pem',
  // specify the scopes you which to access
  scopes: ['https://www.googleapis.com/auth/drive.readonly']
}, function (err, token) {

  console.log(token);

});
```

### Specifying options

The following options can be specified in order to generate the JWT:

```javascript
var options = {

  // the email address of the service account (required)
  // this information is obtained via the API console
  email: 'my-service-account@developer.gserviceaccount.com',

  // an array of scopes uris to request access to (required)
  // different scopes are available for each application (refer to the app documentation)
  scopes: [...],

  // the cryptographic key as a string, can be the contents of the PEM file
  // the key will be used to sign the JWT and validated by Google OAuth
  key: 'KEY_CONTENTS',

  // the path to the PEM file to use for the cryptographic key (ignored is 'key' is also defined)
  // the key will be used to sign the JWT and validated by Google OAuth
  keyFile: 'path_to/key.pem',

  // the duration of the requested token in milliseconds (optional)
  // default is 1 hour (60 * 60 * 1000), which is the maximum allowed by Google
  expiration: 3600000,

  // if access is being granted on behalf of someone else, specifies who is impersonating the service account
  delegationEmail: 'email_address'
};
```

More information:
[https://developers.google.com/accounts/docs/OAuth2ServiceAccount#formingclaimset](https://developers.google.com/accounts/docs/OAuth2ServiceAccount#formingclaimset)

## Compatibility

+ Tested with Node 0.8
+ Tested on Mac OS X 10.8

## Dependencies

+ request

## License

The MIT License (MIT)

Copyright (c) 2013, Nicolas Mercier

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
