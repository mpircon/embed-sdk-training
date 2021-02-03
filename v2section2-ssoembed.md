## Section 2: Authentication & SSO URL

> ### Topics Covered:
> * Overview of Seamless Authentication (SSO Embed)
> * Option 1: Server Script
> * Option 2: API Endpoint (Recommended)

Authentication for an "embedded user" works by creating a special Looker URL that you will use in an iframe. The URL contains the information you want to share, the ID of the user in your system, and the permissions you want that user to have. You’ll then sign the URL with a secret key provided by Looker. There are two ways you can facilitate URL generation:
1. **URL Generation Script** running on the application/ client side server
2. As of Looker 6.22, **Looker's API** allows for creating an SSO URL

These are two methods to generate essentially the same output... a signed URL.

### Client Server Script

This requires storing the shared Embed Key on your application server to sign the URL.

We are already configured for the client server script. Let's see how this is working and how this will simulate what your application backend might do:
* Go to the `webpack-devserver.config.js` and find the `before` statement. As you'll see, when you start up your server, it's serving up the `/auth` endpoint which requests a signed URL (`createSignedUrl`)
* If you select on the definition of `createSignedUrl` (also in `auth_utils`), you'll see it's generating a one-time use, signed URL
* Back at the top of our webpack config, you'll see it's referencing the permissions of the demo user defined in the `demo_user.json during set-up


See [examples](https://github.com/looker/looker_embed_sso_examples)

### Looker API Endpoint

The second (and **preferred**) methodology is using the Looker API endpoint. This does not require the embed secret as an input (a secret does need to be active on the Looker instance) rather it requires an API user with admin privileges [Create SSO Embed URL](https://docs.looker.com/reference/api-and-integration/api-reference/v3.1/auth#create_sso_embed_url). Additionally, it's completely programmatic, easier to build and debug and runs on the Looker server.

This requires some set-up.

To get started, let’s install the Looker SDK. For more information about the Looker API client SDK [see here](https://docs.looker.com/reference/api-and-integration/api-sdk)
* Install the Looker SDK (in this example, we’re going to use `yarn`)
```
yarn add @looker/sdk
```

> <img src="https://cdn3.iconfinder.com/data/icons/basicolor-signs-warnings/24/182_warning_notice_error-512.png" height="25" /> **Issues in the Code - Install Fix**
>  ```
>  yarn add @types/request-promise-native --dev
>  ```

* Next, configure the SDK for your Looker server. Head over to your Looker instance and create your [API 3 keys](https://docs.looker.com/admin-options/settings/users#API3keys).
* Navigate to your .env and fill in your API ID and secret in `LOOKERSDK_CLIENT_ID`, `LOOKERSDK_CLIENT_SECRET`.

>  <img src="https://cdn3.iconfinder.com/data/icons/basicolor-signs-warnings/24/182_warning_notice_error-512.png" height="25" /> **Bug in SDK**
> Issues with .env
> ```
>  ## put Client ID in LOOKERSDK_VERIFY_SSL
>  ## put Client Secret in LOOKERSDK_TIMEOUT
>  LOOKERSDK_VERIFY_SSL=
>  LOOKERSDK_TIMEOUT=
>  ```
>  Also, in the `tsconfig-server.json` udpate/add:
>  ```
>  "strict": false,
>  "esModuleInterop": true
>  ```

Awesome! Now that the `.env` is set-up, let’s set up the environment to get ready for making API calls in the browser.

**First**, let's create an endpoint that will request a URL from the API. Navigate to your `webpack-devserver.config.js` and replace the code beneath `// Authenticate the request is from a valid user here` with the following:

      ```
        // Authenticate the request is from a valid user here
        const src = req.query.src;
        const url = await apiSignedUrl(src, user)
        res.json(url);
      ```

Then import the function we need, `apiSignedUrl`, into webpack-devserver.config.js as well. Place this line near the top:

```
  var { apiSignedUrl } = require('./server_utils/auth_utils')
```


**Next**, navigate to the auth_utils file where we'll want to do two things:

1. We haven't setup the demo environment to get ready for making API calls in the browser so let's import our new typescipt/javascript SDK

```
import {
  Looker40SDK
} from '@looker/sdk'

import { NodeSession, NodeSettings } from '@looker/sdk-rtl'

const settings = new NodeSettings('LOOKERSDK')
const session = new NodeSession(settings)
const sdk = new Looker40SDK(session)
```

2. Create the function that will generate the signed URL

```
export async function apiSignedUrl (url: string, user: any) {
  let signed = await sdk.ok(sdk.create_sso_embed_url({
    target_url: `${process.env.LOOKER_HOST_URL}${url}`,
    ...user
  }))
  return signed
}
```


### Verifying and Debugging

Let's verify that the access token is working by making an API call from the frontend and checking the results of it.

After our dashboard loads, we will want to make an API call and check the user's credentials. Navigate back to `demo.ts`.  Let's make our first API call by inserting this at the bottom of the `setupDashboard()` function, right before the last closing curly brace:

```
const me = await sdk.ok(sdk.me())
console.log('me', me)
```

This is the first time we're making an API call in the application so this is flow when the page reloads

1. The iframe connects and loads, then `setupDashbaord()` is run
2. We make an API call with the JS SDK to grab the current user's info `me()`
3. The JS SDK realizes it doesn't have an accessToken yet, so it will auto-login via the `getToken()` function we created above.
4. `getToken()` goes to the backend, logs into the API, retrieves an access_token for that user and returns to the browser
5. The SDK now has an access_token and will try to run `me()`.
6. We use `console.log(me)` to output it to the console in your browser; open up your developer tools `View > Developer > Javascript Console` or `Cmd+Option+J` to see this output.

You should see the name of your user in the Javascript console. console.log is an easy way to check the value of a variable while debugging or trying to understand what's happening in your code. There are other, more sophisticated ways like [breakpoints](https://developers.google.com/web/tools/chrome-devtools/javascript/breakpoints), but `console.log()` is more than adequate for now.
