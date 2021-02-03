## Section 3: Product & Design Basics

> ### Topics Covered:
> * Custom Filters
> * Customize Formatting of IFrame in Page


### Custom Filters
***

#### Default Filters on Load

Let's start with the current State Filter included in the Embed SDK Demo. First, let's add a variable for the State Filter and import that.

**Add variable** `demo_config.ts`:
```
// the name of the filter on the dashboard for state
export const dashboard_state_filter = 'State'
```

**Import in** `demo.ts`:
```
import {  dashboard_state_filter } from './demo_config'
```

Now, let's find `// Add a listener to the state selector and update the dashboard filters when changed`. Ultimately, this is doing exactly as the comment describes... we are adding a listener that will update the filter in the iframe when a user selects a new filter on the parent page.

Let's refresh our embedded dashboard and change the filter value in the dropdown. Voila! You'll notice the filter update in the iframe.

Note: Make sure you're referencing the same name of the filter in the listener as well as building of the embedded dashboard under `.withFilters` under the `LookerEmbedSDK.createDashboardWithId(dashboardId)`.


#### Customize: Auto-run on Filter Changes

What if we don't want users to have to hit the run button after each change to a filter?

Within our filter listener `const stateFilter...`, after the `dashboard.updateFilters` line, insert a new line and paste `dashboard.run()`

Now when you  update the filters, you'll notice the dashboard will execute without any intervention from the user.


#### API to Populate Filters

Alright, let's start to build on top of our filter example. Let's say we don't want to hardcode our filter options. Instead, we want filter options to prepopulate based on our data. Let's start play with the API!

Remove the options from your dropdown in the HTML (in `index.html`). You're going to want to remove the lines of code the include values for California, Illinois and Minnesota (but `<option value="">Select...</option>`).

**Next**, we're going to be running a query from within the frontend.  Let's create a few variables that we'll use to make the API call.  Place this at the bottom of `demo_config.ts`:

Place this with the other imports near the top of `demo.ts` (Note: Check that the parameters align with values in your model):

```js
export const query_object = {
  "model": "thelook",
  "view": "order_items",
  "fields": ["users.state"],
  "sorts": ["users.state asc"],
  "filters": {}
}
// field name we will want to extract for dropdown
export const query_field_name = 'users.state'
```

Place this with the other imports near the top of `demo.ts`:

```
import {  query_object, query_field_name } from './demo_config'
```

***

<img src="https://cdn3.iconfinder.com/data/icons/basicolor-signs-warnings/24/182_warning_notice_error-512.png" height="25" /> **Requires Updated Code - Errors using code form [SKO Training](https://saleseng.dev.looker.com/projects/embed-sdk-sko-markdown/files/section4.md)**


At the bottom of demo.ts we will add a function that creates new `<option/>` HTML tags for us. We will send the function the list of states, and it will return the options for us.

```js

function addStateOptions(data: any) {
  const dropdown = document.getElementById('select-dropdown')
  data.forEach(function (row: any, i: number) {
    if (query_field_name && row[query_field_name] && dropdown) {
      const new_option = document.createElement('option')
      new_option.value = row[query_field_name]
      new_option.innerHTML = row[query_field_name]

      // replace or create the dropdown item
      if (dropdown.children[i+1]) {
        dropdown.children[i+1].replaceWith(new_option)
      } else {
        dropdown.appendChild(new_option)
      }
    }
  })
}
```

We have to get the list of states so let's run an API call that grabs all states and passes it to our new function.  We want to run this after the dashboard loads, so place it on a new line after `const me = await sdk.ok(sdk.me())` within the function `setupDashboard(...)`

```js
  const states = await sdk.ok(sdk.run_inline_query(
    {
      body: query_object,
      result_format: 'json'
    }
  ))
  addStateOptions(states)
```

<img src="https://cdn3.iconfinder.com/data/icons/basicolor-signs-warnings/24/182_warning_notice_error-512.png" height="25" /> **End of Errors**

***

#### Customize: Applying Custom Formatting

What if we want to change the CSS, colors, font to be inline in our brand?

Looker utilizes **Themes** which customizes the appearance of your embedded Looker dashboards and Explores. Themes can customize font family, text color, background color, button color, tile color, and other visual elements.

First, create a new theme under the **Themes** page in the **Platform** section of the **Admin** panel

To implement, add this above `.build()`:

```
.withTheme('theme1')
```

Bonus: Add in a special theme for this user's external group_id. Change `withTheme()` to

```
.withTheme( (user.external_group_id == 'group2') ? 'theme1' : 'theme2' )
```
The logic above says:
> If external\_group_id is equal to group2, then display theme1, else use theme2

Feel free to revert it :)


### Customization of iFrame in Parent Page
***

Another common request is to make sure that multiple scrollbars don't appear:  one within the iframe and one on the parent page.

![HTML height](https://bryan-at-looker.s3.amazonaws.com/images/embed-sdk-sso/section2-height-scroll-before.png)

In order to do this, we need to listen to metadata that Looker is sending out to the parent page through Javascript events. The `page:properties:changed` event ([docs](https://docs.looker.com/reference/embedding/embed-javascript-events#page:properties:changed)) gives us the ability to listen to the height of the Looker iframe and then we can dynamically change the style of the parent `div, #dashboard`.

The `page:properties:changed` event looks like this:

```
{
  "type": "page:properties:changed",
  "height": 2444,
  "dashboard": {
    "id": 2,
    "title": "Business Pulse",
    "dashboard_filters": {
      "Date": "30 days",
      "State": "California"
    },
    "absoluteUrl": "https://sko2020.dev.looker.com/embed/dashboards/2?embed_domain=https:%2F%2Fembed.demo:8080&sdk=2&State=California&theme=sko&Date=30%20days&filter_config=%7B%22Date%22:%5B%7B%22type%22:%22past%22,%22values%22:%5B%7B%22constant%22:%2230%22,%22unit%22:%22day%22%7D,%7B%7D%5D,%22id%22:0%7D%5D,%22State%22:%5B%7B%22type%22:%22%3D%22,%22values%22:%5B%7B%22constant%22:%22California%22%7D,%7B%7D%5D,%22id%22:1%7D%5D%7D",
    "url": "/embed/dashboards/2?embed_domain=https:%2F%2Fembed.demo:8080&sdk=2&State=California&theme=sko&Date=30%20days&filter_config=%7B%22Date%22:%5B%7B%22type%22:%22past%22,%22values%22:%5B%7B%22constant%22:%2230%22,%22unit%22:%22day%22%7D,%7B%7D%5D,%22id%22:0%7D%5D,%22State%22:%5B%7B%22type%22:%22%3D%22,%22values%22:%5B%7B%22constant%22:%22California%22%7D,%7B%7D%5D,%22id%22:1%7D%5D%7D"
  }
}
```

First let's add a new function at the bottom of `demo.ts` that will listen to the event and apply the change to the height of the element that contains the iframe:

```
function changeHeight( event: any ) {
  console.log(event)
  const div = document.getElementById('dashboard')
  if (event && event.height && div) {
    div.style.height = `${event.height+15}px`
  }
}
```

We'll get a little bit of a buffer by adding 15 pixels.

Then within the Embed SDK code (still in our `demo.ts` file) we need to tell it to perform the function below when it receives the `page:properties:changed` event. Copy and paste this function on its own line above `.build()`:

```
.on('page:properties:changed', changeHeight )
```


![HTML height](https://bryan-at-looker.s3.amazonaws.com/images/embed-sdk-sso/section2-height-html.png?raw=true)

The scroll bar is now on the outer page and not in the iframe, it feels more native and prevents the "multiple scroll bar problem".
![HTML height](https://bryan-at-looker.s3.amazonaws.com/images/embed-sdk-sso/section2-height-scroll-after.png)
