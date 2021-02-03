## Authentication & SSO URL

> ### Topics Covered:
> * Overview of Seamless Authentication (SSO Embed)
> * Option 1: API Endpoint (Recommended)
> * Option 2: Server Script

Authentication for an **Embedded User** works by creating a one-time use, signed Looker URL that you will use in an iframe. The URL contains the information you want to share, the ID of the user in your system, and the permissions you want that user to have. To generate the URL, the **Looker API** includes a `create_sso_embed_url` endpoint, which takes a set of embed parameters that includes the URL of the content you want to embed, and returns a complete, encoded, cryptographically signed SSO URL.

Another option is to have a  **URL Generation Script** running on the application side server. We will outline this method but it is not preferred as it can be more complicated and harder to debug.

### Looker API Endpoint

As discussed, Looker has the `create_sso_embed_url` endpoint to generate a signed, SSO embed URL. This is the preferred methodology for generating the iframe URL and authenticating embedded users as it is completely programmatic, easier to build and debug and runs on the Looker server. Requirements include:
* API user with Admin privileges
* Embed secret is NOT required as an input however, a secret needs to be active on your Looker Instance

For best practices around managing API credentials see [**here**](https://docs.looker.com/reference/api-and-integration/api-auth#managing_api_credentials)

We provided our API credentials to the server during set-up so when you started up the demo during Section 1, you were already using Looker's SSO Embed to generate the iframe! Let's talk through a couple components to show how this might simulate your backend.

**First**, navigate to the `demo_user.json` file. This determines the necessary parameters we need to pass to Looker about the user and their permissions and including user ID, data and feature access, session length, etc.


**Next**, let's see what's happening when you start up the server. Navigate to your `webpack-devserver.config.js` and find the code beneath `// Authenticate the request is from a valid user here`. Ultimately, this is requesting a signed URL to generate the iframe. If you "Go To Definition"  on `apiSignedURL` and view the function, you'll see the function requesting a one-time use, signed URL via Looker's API (`create_sso_embed_url` endpoint). Additionally, it is utilizing the permissions outlined in the `demo_user.json` to authenticate the "user".


        // Authenticate the request is from a valid user here
        const src = req.query.src;
        const url = await apiSignedUrl(src, user)
        res.json(url);


  Definition `apiSignedURL`:


      export async function apiSignedUrl (url: string, user: any) {
        let signed = await sdk.ok(sdk.create_sso_embed_url({
          target_url: `${process.env.LOOKER_HOST_URL}${url}`,
          ...user
        }))
        return signed
      }



### Client Server Script

As mentioned above, another option for generating the signed URL is to have script running on the application server. This requires storing the shared Embed Key on your application server to sign the URL.

For this, it would require adding the Embed Secret to the `LOOKER_EMBED_SECRET=` parameter in the URL. The embed secret grants access to all of your data, so note the following:

* Do not share your secret with anyone you do not want to have complete access to your instance.
* Do not reset your secret if you already are using it in another context.
* Your code should never store the secret in the web browser.


For examples of the SSO generation script in various languages, see [**here**](https://github.com/looker/looker_embed_sso_examples)
