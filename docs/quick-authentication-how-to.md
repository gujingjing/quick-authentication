# Sign in Microsoft account users to single-page apps with Microsoft Quick Authentication

Microsoft Quick Authentication is a lightweight JavaScript library that makes it easier to add **Sign in with Microsoft** support to a single-page app (SPA). Quick Authentication uses the  Microsoft Authentication Library for JavaScript (MSAL.js) to handle authentication and authorization for users with personal Microsoft accounts from Outlook, OneDrive, Xbox Live, and Microsoft 365.

> Microsoft Quick Authentication is in public preview. This preview is provided without a service-level agreement and isn't recommended for production workloads. Some features might be unsupported or have constrained capabilities. For more information, see [Supplemental terms of use for Microsoft Azure previews](https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/).

## How it works

To enable Quick Authentication, you first add a reference to the library in a `<script>` element, then an HTML or JavaScript snippet that displays the sign-in button on the page. When a user selects the button, they're presented with a sign-in prompt. You can configure both the button and the prompt.

All browsers support a configurable sign-in button and prompt provided by Quick Authentication called the _standard sign-in experience_. Microsoft Edge supports an _enhanced sign-in experience_, which is also configurable.

- **Standard sign-in experience** - Quick Authentication shows the _standard_ sign-in button and prompt to users not signed in to Microsoft Edge or that are using a another browser.

  ![Standard sign-in button showing the Sign in with Microsoft text label](./media/large.png)

- **Enhanced sign-in experience** (Microsoft Edge desktop platforms _only_) - Microsoft Edge users who've signed-in to the browser with their personal Microsoft account are shown a personalized sign-in button and prompt (also customizable).

    In this public preview, [Microsoft Edge](https://www.microsoft.com/en-us/edge?r=1) is required for the enhanced experience.

  ![Enhanced sign-in button showing personalized text label](./media/large-known.png)

  Here's an example of sign-in prompt in MSA profile in Edge: ![Enhanced sign-in prompt](./media/signin-prompt.png)

## Browser support

Unless otherwise noted, browser compatibility applies only to the two most recent (current and previous) stable supported versions of the specified browser.

| Browser             | Standard <br/>sign-in experience                                      | Enhanced <br/>sign-in experience                                      |
|---------------------:|:----------------------------------------------------------------------:|:----------------------------------------------------------------------:|
| Firefox             | ![yes](./media/common/yes.png) | ![no](./media/common/no.png)  |
| Google Chrome       | ![yes](./media/common/yes.png) | ![no](./media/common/no.png)  |
| Microsoft Edge (Windows, Mac, Linux)      | ![yes](./media/common/yes.png) | ![yes](./media/common/yes.png)  |
| Microsoft Edge (Other platforms)     | ![yes](./media/common/yes.png) | ![no](./media/common/no.png)  |
| Safari              | ![yes](./media/common/yes.png) | ![no](./media/common/no.png)  |

## Register your application

First, complete the steps in [Register an application with the Microsoft identity platform](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app) to register your single-page application.

Use the following settings for your app registration. **Bold text** in the table represents a UI element in the Azure portal and `code-formatted text` indicates a value you enter.

| App registration setting    | Value                                                                  | Notes                                                     |
|----------------------------:|:-----------------------------------------------------------------------|:----------------------------------------------------------|
| **Name**                    | `SPA with Quick Auth`                                                  | Suggested value. You can change the app name at any time. |
| **Supported account types** | **Personal Microsoft Accounts Only**                                   | Required value for Microsoft Quick Authentication.        |
| **Platform type**           | **Single-Page Application**                                            | Required value for Microsoft Quick Authentication.        |
| **Redirect URI**            | `http://localhost:<port>/blank.html` or `https://<your-tenant-id>/blank.html` | Recommended for Microsoft Quick Authentication.           |

## Add the Quick Authentication script

Next, add a `script` element that adds the Quick Authentication library reference (_ms\_auth\_client.min.js_) to your application. It's safe to add the `defer` attribute to load the script asynchronously.

```html
<script async defer
        src="https://edge-auth.microsoft.com/js/ms_auth_client.min.js"
        ></script>
```

## Choose an integration method: HTML or JavaScript

There are two ways to add Quick Authentication to your app:

- **HTML**: If have access to the application's HTML, [add the sign-in button in HTML](#option-1-add-sign-in-button-via-html).
- **JavaScript**: If you don't have access to the HTML, [add the sign-in button with JavaScript](#option-2-add-sign-in-button-via-javascript).

## Option 1: Add sign-in button via HTML

If you have access to adjust your application's HTML, this approach offers a declarative way to configure Quick Authentication.

1. Configure the library

    Add the configuration `div` to your page:

    ```html
    <div id="ms-auth-initialize"
         data-client_id="<your application client ID>"
         data-callback="<your callback function>"
         data-login_uri="<your application redirect URI>">
    </div>
    ```

    - Set `data-client_id` to the Application (client) ID of the application registration.
    - Set `data-callback` to the name of the JavaScript function you'd like called when a user is authenticated
    - Set `data-login_uri` to the redirect URI you entered in your application registration in the Azure portal. If this value isn't set, then for application https://abc.com, Quick Authentication library will use https://abc.com/blank.html as the default.
    - `data-callback` should be a string, which represents a function, which is present in JavaScript on the web-page. Else initialization will fail.

    You'll need to write JavaScript code to receive the authentication event and handle completing the registration and sign-in processes. We'll discuss how to do that, below.

1. Inject the sign-in button

    Place a `div` in your authentication component to indicate where Quick Authentication should render the button:

    ```html
    <div class="ms-auth-button"></div>
    ```

For details about modifying the sign-in button's appearance and the prompt's appearance and behavior, see the [Quick Authentication library reference](./quick-authentication-reference.md).

## Option 2: Add sign-in button via JavaScript

Use this approach if you don't have the ability to modify the application's HTML.

1. Configure the library

    Configuration of the Quick Authentication library can safely happen when the page has finished loading, by calling `ms.auth.initialize` and passing it a configuration object:

    ```javascript
    window.onload = function () {
      const result = ms.auth.initialize({
        client_id: '<your application client ID>',
        callback: <your callback function>,
        login_uri: '<your application redirect URI>',
        cancel_on_tap_outside: false
      });
      if (result.result == 'success') {
        // Proceed.
      } else {
        // Initialization failed.
        console.error('ms.auth.initialize failed: ', result);
      }
    };
    ```

    - Set `client_id` to the Application (client) ID of the application registration.
    - Set `callback` to the name of the JavaScript function you'd like called when a user is authenticated.
    - Set `login_uri` to the redirect URI you entered in your application registration in the Azure portal. If this property isn't set, then for application https://abc.com, Quick Authentication library will use https://abc.com/blank.html as the default.

    If `ms.auth.initialize` succeeds, it returns `{result: 'success'}`. If `ms.auth.initialize` fails, it returns `{result: 'failure', reason: <string-explaining-why-it-failed>}`.

    The following are some possible reasons for failure:

    - Initialization was called more than once. Here API returns `{result: 'failure', reason: 'Library already initialized'}`.
    - `callback` isn't a valid function or `client_id` isn't set. In these cases, API returns `{result: 'failure', reason: 'Invalid configuration'}`.

    You'll need to write JavaScript code to receive the authentication event and handle completing the registration and sign-in processes. We'll discuss how to do that, below.

    [Quick Authentication can be configured](./quick-authentication-reference.md) to allow more control over the behavior of the library.

1. Inject the button

    To render a sign-in button, call the following JavaScript just after initializing the library (shown above):

    ```javascript
    ms.auth.renderSignInButton(
      document.getElementById('<id of parent element>'),
      {} // Using default options.
    );
    ```

    You'll need to identify the DOM node that you want the library to insert the button into. If DOM node is `null` or `undefined` because it wasn't found, then this API will throw exception.

    Quick Authentication allows you to [customize the look and feel of the button](./quick-authentication-reference.md#customizing-the-sign-in-button) to better align with your application's design. We'll discuss more about customizing the look and feel of the button, below.

## Responding to authentication events

When an authentication event occurs, the Quick Authentication library calls the `callback` function you specified in the configuration, passing in an [AccountInfo](./quick-authentication-reference.md#data-type-accountinfo) object containing the user's account details as a parameter:

```javascript
function myCallback(sign_in_account_info) {
  console.log(sign_in_account_info);
  // Your sign in/up logic
}
```

The [SignInAccountInfo](./quick-authentication-reference.md#data-type-signinaccountinfo) object includes the following properties:

- `fullName` - the user's full name
- `username` - the user's email address or phone number
- `photoUrl` - a base64-encoded dataURI representing the user's avatar
- `id` - a GUID that is unique across all Microsoft accounts
- `idToken` - [ID token](https://docs.microsoft.com/en-us/azure/active-directory/develop/id-tokens) received during sign-in process

We recommend using the `id` as a key, rather than the email address, because an email address isn't a unique account identifier. It's possible for a user to use a Gmail address for a Microsoft Account, and to potentially create two different accounts (a Microsoft account and a Google account) with that same email address. It's also possible for a Microsoft Account user to change the primary email address on their account.

## Signing out

Use `ms.auth.signOut()` to sign out a user:

```javascript
ms.auth.signOut(username, function (result) {
  if (result.result) {
    // Finish the logout process in your application.
  } else {
    console.log('Sign out failed, error: ', result.error);
  }
});
```

You'll need to pass in `username` field of `AccountInfo` object you received as part of the authentication event and a callback to receive the result of the `signOut()` call. The argument returned to the callback will be an object with the following structure:

- `result` - `true` if the user was signed out, `false` if not
- `error` - if `result==false`, will be an object with a `message` property explaining the error

## Advanced: Querying authentication status

You can determine if the current user is signed-in by calling `ms.auth.startGetCurrentAccount()` and passing in the name of a callback function that is ready to receive the account information:

```javascript
function accountCallback(account_info) {
  if (account_info) {
    // Use account.
  }
}

ms.auth.startGetCurrentAccount(accountCallback);
```

When called, `startGetCurrentAccount()` will trigger the callback and pass in a value. If the user is signed-in, the argument will be an [AccountInfo](./quick-authentication-reference.md#data-type-accountinfo) object. If the user isn't signed-in, the argument will be `null`.

## Customizing the Quick Authentication experience

To fully control your users' authentication experience, build and maintain your own button to trigger the sign-in flow.

### Start the sign-in process

To begin the sign-in process when a user activates your custom button, you'll need to call `ms.auth.startSignIn()`:

```javascript
button.addEventListener('click', function() {
  ms.auth.startSignIn();
});
```

As with the previous examples, successful authentications will be routed into the callback you defined when initializing the library.

### Show the sign-in prompt

If you want to control the display of the enhanced sign-in prompt in Microsoft Edge, use the `ms.auth.prompt()` method:

```javascript
ms.auth.prompt('right', function(notification) {
  const reason = notification.reason;
  if (notification.type === 'display' && !notification.displayed) {
    if ( reason === 'browser_not_supported' ) {
      console.log('prompt not supported in browser');
    }
  } else if (notification.type === 'skipped') {
    if (reason === 'user_cancel') {
      console.log('user cancelled');
    }
  } else if (notification.type === 'dismissed') {
    if (reason === 'credential_returned') {
      console.log('Got sign-in credentials');
    }
  }
});
```

This method may be called with no arguments to use the default configuration. You may also supply the following arguments to customize its appearance and/or respond to specific user interactions with the prompt.

The first argument is the position. By default, the prompt is rendered on the left. You can position it "center" or "right" as well.

The second argument is a callback function that receives notifications from the prompt, based on user interactions. The callback receives a [PromptMomentNotification](./quick-authentication-reference.md#data-type-promptmomentnotification) object when the user interacts with it.

### Close the sign-in prompt

If you want to close the enhanced sign-in prompt in Microsoft Edge, use the `ms.auth.cancel()` method:

```javascript
ms.auth.cancel();
```

### Exponential cool-down for the sign-in prompt

If the user closes the sign-in prompt manually, the sign-in prompt is suppressed. A user closes sign-in prompt when they tap Close (**X**) in the top-right corner of the prompt or when they close it by pressing Escape key. When the user closes the sign-in prompt, it won't display in the same browser for that application for sometime. This duration is called the cool-down period.

| Consecutive closures of <br/>sign-in prompt | Cool-down period <br/>(sign-in prompt disabled) |
|:-------------------------------------------:|-------------------------------------------------|
| 1                                           | Two (2) hours                                   |
| 2                                           | One (1) day                                     |
| 3                                           | One (1) week                                    |
| 4+                                          | Four (4) weeks                                  |

The cool-down status will reset after a successful sign-in.

## Working with access tokens

Quick Authentication surfaces the access token request APIs in MSAL.js. For details and example code for calling these methods, see [Microsoft Quick Authentication configuration and JavaScript API reference](./quick-authentication-reference.md).

Call these methods in the following order to get an access token:

1. Call [ms.auth.getMSALAccount](./quick-authentication-reference.md#method-msauthgetmsalaccount).
1. Attempt to to get an access token silently with [ms.auth.acquireTokenSilent](./quick-authentication-reference.md#method-msauthacquiretokensilent).
1. If the silent call fails, call [ms.auth.acquireTokenPopup](./quick-authentication-reference.md#method-msauthacquiretokenpopup) to display the sign-in window.

## Logging

To help in troubleshooting and debugging errors in your apps that use Microsoft Quick Authentication, enable event logging.

To enable Quick Authentication event logging, add the [`autoLogEvents`](./quick-authentication-reference.md#logging-with-autologevents) query parameter when you include the _ms\_auth\_client.min.js_ script.

```html
<!-- Enable logging by adding "autoLogEvents" query param -->
<script async defer
        src="https://edge-auth.microsoft.com/js/ms_auth_client.min.js?autoLogEvents=1"
        ></script>
```

To also enable logging for MSAL.js, add the [`logMsalEvents`](./quick-authentication-reference.md#msal-logging-with-logmsalevents) query parameter in addition to `autoLogEvents`:

```html
<!-- Enable MSAL.js logging by adding both "autoLogEvents" and "logMsalEvents" query params -->
<script async defer
        src="https://edge-auth.microsoft.com/js/ms_auth_client.min.js?autoLogEvents=1&logMsalEvents=4"
        ></script>
```

## Troubleshoot specific errors

To ease your troubleshooting and debugging efforts, start by enabling event logging as described in [Logging](#logging).

### unauthorized_client

If during sign-in flow, your application runs into following error:

```console
We're unable to complete your request

unauthorized_client: The client does not exist or is not enabled for consumers. If you are the application developer, configure a new application through the App Registrations in the Azure Portal at https://go.microsoft.com/fwlink/?linkid=2083908.
````

Then check your application registration and in "Overview" tab check the value of "Supported account types". If it's not "Personal Microsoft account user", then this error can show up.

To fix this issue, you can "Delete" your application registration and recreate a new one with "Supported account types" set to "Personal Microsoft account user".

### Running into "redirect_uri not valid" error

If during sign-in flow, your application runs into following error:

```console
We're unable to complete your request

invalid_request: The provided value for the input parameter 'redirect_uri' is not valid. The expected value is a URI which matches a redirect URI registered for this client application.
````

Then check your application registration and in "Authentication" tab check the value in "Platform configurations". If "Single-page application" isn't present as a platform and it's "Redirect URIs" aren't specified, then this error can show up.

To fix this issue, select "Add a platform". Then select "Single page application". Then add "Redirect URIs". If your application is "https://www.myapplication.com", we recommend setting "https://www.myapplication.com/blank.html" and serve a blank webpage from that location.

Also, **don't select** any option under "Implicit grant and hybrid flows".

### Callback function not getting called after successful sign-in

If after completing sign-in flow the JavaScript callback function isn't getting called, then check the initialization code.

If div `"ms-auth-initialize"` is used to initialize Quick Authentication library, check whether the `data-callback` argument has been set correctly. For a JavaScript callback function like below:

```javascript
function handleSignInResponse(accountInfo) {
  // Use accountInfo.
}
```

The correct way of setting callback is shown in the following snippet:

```html
<div id="ms-auth-initialize" data-client_id="<YOUR-APPLICATION-ID>" data-callback="handleSignInResponse">
</div>
```

The following snippet is incorrect:

```html
<div id="ms-auth-initialize" data-client_id="<YOUR-APPLICATION-ID>" data-callback="handleSignInResponse()">
</div>
```

Notice the `()` at the end, which causes the problem.

## Next steps

For more details about prompt configuration and API reference, see [Microsoft Quick Authentication configuration](./quick-authentication-reference.md).
