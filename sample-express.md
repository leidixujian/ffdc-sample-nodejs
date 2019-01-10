# Node.JS Sample App

The purpose of the sample project is to show you how to integrate a
machine authentication flow with the **Fusion**Fabric.cloud
Authentication Service, and call an API from **Fusion**Fabric.cloud,
with the identity of the app.

You will implement the OAuth2 Client Credentials Grant flow, shown
below.

![The OAuth2 Autorization Client Credentials Grant
flow.](img/auth-client-creds-grant.png)

To find out more about the Client Credentials Grant, see
[RFC6749](https://tools.ietf.org/html/rfc6749#section-4.4).

### Get it from GitHub

If you want to go ahead, and you have a GitHub account, clone the
repository and start working with it. The link is:

> <https://github.com/FusionFabric/ffdc-sample-nodejs>

## Prerequisites

For this application, you will use [Express](https://expressjs.com/), a
web framework for Node.js. Therefore, you must have
[Node](https://nodejs.org/en/download/) and
[npm](https://www.npmjs.com/get-npm) installed on your computer.

You must also create an app on the **Fusion**Fabric.cloud Developer
Portal that include APIs from Financial Toolbox.

## Bootstrap App

1.  Create a working directory and open it in terminal or command line:

<!-- end list -->

``` notoggle
mkdir ffdc-node-sample && cd ffdc-node-sample
```

2.  Create a `package.json` file for your application:

<!-- end list -->

``` notoggle
npm init
```

You can accept the defaults for all the options that are prompted on the
terminal.

3.  Install the required dependencies:

<!-- end list -->

``` notoggle
npm install --save openid-client dotenv node-fetch request express
```

4.  Create a sudirectory, named `src`, and open it.

<!-- end list -->

``` notoggle
mkdir src && cd src
```

5.  Create your main app file, an empty text file named `index.js`. In
    the following sections you will add the required code to this file.

<!-- end list -->

``` notoggle
touch index.js
```

6.  In the root directory, Open `package.json` and replace the `main`
    and `scripts` sections with the following:

<div class="headed-code" data-header="package.json">

``` js notoggle numberLines
{ 
// ...

  "main": "src/index.js",
  "scripts": {
    "start": "node -r dotenv/config src/index.js",
    "watch-node": "nodemon src/*",
    "serve-debug": "nodemon --inspect src/index.js"
  },

// ...
}
```

</div>

## OpenID Service

You need to define a helper module to handle the connection to the
**Fusion**Fabric.cloud Authentication service. You will

<span class="text-title">**To configure the OpenID service**</span>

1.  In the `src` directory, create a text file, named `openIdIssuer.js`,
    and add the following code:

<div class="headed-code" data-header="src/openIdIssuer.js">

``` js notoggle numberLines
const openIdClient = require("openid-client");
 
module.exports = function (discoveryType = "auto") {
    return openIdClient.Issuer.discover(process.env.AUTHORIZATION_WELLKNOWN);
}
```

</div>

This module uses the
[`discover`](https://www.npmjs.com/package/openid-client#via-discovery-recommended)
function of **openid-client** to connect to the Discovery service of the
portal.

2.  At the root of the application, create a text file named `.env`

<!-- end list -->

``` notoggle
cd .. && touch .env
```

3.  Add the following code, replacing the tokens with the corresponding
    values:

<div class="headed-code" data-header=".env">

``` notoggle numberLines
CLIENT_ID= "{YOUR-CLIENT-ID}"
CLIENT_SECRET= "{YOUR-CLIENT-SECRET}"
AUTHORIZATION_WELLKNOWN = "{WELLKNOWN-OPENID-CONFIG-URL}"
```

</div>

In the `.env` configuration file you store the variables that are passed
to the `openIdClient` module to retrieve the authorization token. These
are the following:

  - The `CLIENT_ID` and `CLIENT_SECRET` are generated for your app in
    the Developer Portal.
  - The `AUTHORIZATION_WELLKNOWN` is the URL of the Discovery service,
    described in the [Discover](#discovery-service) section.

## Main App

In this section you write the main body of your app.

1.  Open `src/index.js` and add the following code:

<div class="headed-code" data-header="src/index.js">

``` js notoggle numberLines
const express = require('express');
const fetch = require('node-fetch');
global.Headers = fetch.Headers;

const issuer = require('./openIdIssuer')();

const app = express();
var port = 5000;

let client;
let token;
```

</div>

2.  Initialize OIDC

<div class="headed-code" data-header="src/index.js">

``` js notoggle numberLines
issuer.then(issuer => {
  client = new issuer.Client({
    client_id: process.env.CLIENT_ID,
    client_secret: process.env.CLIENT_SECRET
  });

  app.listen(port, () => console.log(`Sample app listening on port ${port}!`));
});
```

</div>

With the above code you:

  - instantiate an **openid-client** client
  - you pass the `CLIENT_ID` and `CLIENT_SECRET` of the app that you
    created on Developer Portal, to the authorization server
  - configure your Express app to run on port `${port}`, which is
    `5000`, in this case.

<!-- end list -->

3.  Define the call route

<div class="headed-code" data-header="src/index.js">

``` js notoggle numberLines
app.get('/', async (req, res, next) => {
  // Check for token validity
  var new_token_needed = true;
  if (token !== null && token !== undefined) new_token_needed = token.expired();

  // ask for a token if needed
  if (new_token_needed) {
    token = await client.grant({
      grant_type: 'client_credentials',
      scope: 'openid'
    });
  }
}):
```

</div>

With the above code you define the call route of your app. For each call
to the root endpoint `/` of your **Express** app, the app checks if a
token is already available and valid. If not, it calls the grant flow to
retrieve a new token and store it into the `token` variable.

## API Call

You are now ready to add the call to a **Fusion**Fabric.cloud API. Add
the following code in the `app.get() => {}` block of `index.js`:

<div class="headed-code" data-header="src/index.js">

``` js notoggle numberLines
app.get('/', async (req, res, next) => {

//...

  let countries;
  var url = 'https://api.ffdcdev.fusionfabric.cloud/referential/v1/countries';

  try {
    const response = await fetch(url, {
      method: 'get',
      headers: new Headers({
        Authorization: 'Bearer ' + token.access_token,
        'Content-Type': 'application/x-www-form-urlencoded'
      })
    });

    countries = await response.json();
  } catch (error) {
    console.log(error);
  }

  res.send(countries);

// ...  

});
```

</div>

The code illustrates:

  - The call to an API, in this case, the `GetCountries` endpoint of the
    **Referential Data** API, from **Financial Toolbox**.
  - How to add the access token to the headers of each subsequent call
    of the endpoint defined in `url`.

## Run your App

Your are now ready to run your app.

1.  Start your app:

<!-- end list -->

``` notoggle
$ npm start

...
Sample app listening on port 5000!
```

2.  Start your browser, and go to
    [localhost:5000](#http://localhost:5000/). The list of the
    countries, retrived from the **Referential Data** API of
    **Fusion**Fabric.cloud **Financial Toolbox**, is displayed.

![The countries list displayed by calling a **Fusion**Fabric.cloud API
with the OAuth2 Autorization Client Credentials Grant
flow.](img/auth-client-creds-result.png)
