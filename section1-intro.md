## Section 1: Embed SDK Set-up

### Introduction

The Looker JavaScript Embed SDK is designed to facilitate using Looker embedded content in your web application. The goal is to make communication between a host website and one or more embedded dashboards, looks or explores easier and more reliable.

> ### Topics Covered:
> * Demo Requirements & Set-up
> * Troubleshooting

### Demo

Download/clone the linked [**Repo**](https://github.com/bryan-at-looker/embed-sdk-api) which includes sample code and a simple demo of the embed SDK. Because of Looker’s attention to security, the demo requires a bit of setup.

### Step 1: Enable Embedding in your Looker instance

(This is documented in more detail [**here**](https://docs.looker.com/r/sdk/sso-embed))

* Navigate to **Admin** > **Platform** > **Embed** on your Looker instance. This requires Admin privileges.
* The demo server runs by default at [http://localhost:8080](http://localhost:8080). By adding that address to "Embedded Domain Whitelist" you can enabled the demo to receive messages from Looker.
* Turn on "Embed Authentication"
* Make sure an Embed Secret is enabled on your instance.
* Navigate to **User** page on the instance and generate your API3 keys

### Step 2: Customize the Demo settings for your Looker instance

1. Provide API credentials to the server.

    The Embed SDK uses `dotenv` a common Node package that will look for a `.env` and load them up for you.  For light reading on environment variables you can read [this article](https://medium.com/chingu/an-introduction-to-environment-variables-and-how-to-use-them-f602f66d15fa).

    ```
    mv .env.example .env
    ```

    Update with your Host URL and API ID and secret. Keep the Embed Secret blank. Looker admins can generate an API3 key in the Edit User page, as described on the [Users documentation page](https://docs.looker.com/admin-options/settings/users#api3_keys).

>  <img src="https://cdn3.iconfinder.com/data/icons/basicolor-signs-warnings/24/182_warning_notice_error-512.png" height="25" /> **Bug in SDK**
>
> To get functioning demo:
> ```
>  ## Put Client ID in LOOKERSDK_VERIFY_SSL parameter
>  ## Put Client Secret in LOOKERSDK_TIMEOUT parameter
>  LOOKERSDK_VERIFY_SSL=
>  LOOKERSDK_TIMEOUT=
>  ```


2. Edit the `demo/demo_config.ts` file with the appropriate host and for the pages you want to embed.

    ```javascript
    // The address of your Looker instance. Required.
    export const lookerHost = 'https://customer.looker.com'

    // A dashboard that the user can see. Set to 0 to disable dashboard.
    export const dashboardId = 1
    // A Look that the user can see. Set to 0 to disable look.
    export const lookId = 1
    ```

3. Edit the `demo/demo_user.json` file to be appropriate for the type of user you want to embed.

    ```javascript
    {
      // External embed user ID. IDs are not shared with regular users. Required
      "external_user_id": "user1",
      // First and last name. Optional
      "first_name": "Pat",
      "last_name": "Embed",
      // Duration before session expires, in seconds. Required.
      "session_length": 3600,
      // Enforce logging in with these permissions. Recommended.
      "force_logout_login": true,
      // External embed group ID. Optional.
      "external_group_id": "group1",
      // Looker Group IDs. Optional
      "group_ids": [],
      // Permissions. See documentation for details. Required.
      "permissions": [
        "access_data",
        "see_looks",
        "see_user_dashboards",
      //   see_lookml_dashboards
        "explore",
      //   create_table_calculations
      //   download_with_limit
      //   download_without_limit
      //   see_drill_overlay
      //   see_sql
        "save_content",
        "embed_browse_spaces"
      //   schedule_look_emails
      //   send_to_sftp
      //   send_to_s3
      //   send_outgoing_webhook
      //   schedule_external_look_emails
      ],
      // Model access permissions. Required.
      "models": ["powered_by", "thelook"],
      // User attributes. Optional.
      "user_attributes": { "locale": "en_US" },
      // Access filters. Optional.
      "access_filters": { "powered_by": { "products.brand": "Allegra K" } }
    }
    ```

### Step 3: Build and run the demo

#### Node server

* `npm install`
* `npm start`
* The server will print out what host and port it is running on. If it is different than `http://localhost:8080` then you will need to add that to your Embedded Domain Whitelist.

#### Python server

* `npm install`
* `npm run python`
* The server will print out what host and port it is running on.

You may need to `pip install six` to install the Python 2/3 compatibility layer.

### Troubleshooting

#### Logging

The Embed SDK is built on top of [chatty](https://github.com/looker-open-source/chatty). Chatty uses [debug](https://github.com/visionmedia/debug) for logging. You can enable logging
in a browser console with

```javascript
localStorage.debug = 'looker:chatty:*'
```

Note that both the parent window and the embedded content have separate local storage, so you can enable logging on one, the other or both. You can disable logging with

```javascript
localStorage.debug = ''
```
