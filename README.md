# Ringtail User Interface Extension SDK
This SDK provides an API to communicate with the Ringtail UI for use in UI extensions, part of Ringtail's extensibility model.

## API Documentation
View the [API reference documentation](API.md).

## About User Interface Extensions
You can extend Ringtail by embedding third-party web applications directly into the Ringtail interface. Such external applications are called user interface extensions (UI extensions). Ringtail provides UI extensions with user and workspace context and access to the Ringtail Connect API. This allows UI extensions to read and write data to Ringtail and respond to user actions in the Ringtail UI.

The following diagram shows how an external web application (green) can communicate with Ringtail (blue). The UI Extension SDK (yellow) provides the glue for UI extension clients to communicate directly with Ringtail.

![Ringtail App Communication](https://docs.google.com/drawings/d/e/2PACX-1vQaelod9Flf14CCSyP4MhR4Qznl6n_0EllVzdNiB5gnvsdsYqO5bcwMbTphlMZUbr7tgKqqniZ0HuOx/pub?w=572&h=272)

[Edit the diagram](https://docs.google.com/drawings/d/19RsszUNRVVsDDBWSVHs8ffEncUDB66Hi78pgaAGGkhQ/edit?usp=sharing)

## Install the SDK
The SDK is available as a package on NPM:

`npm install ringtail-extension-sdk`

To support IE11, you also need to provide promise and fetch polyfills, such as:

`npm install promise-polyfill whatwg-fetch`

> NOTE: This library only works in web browsers. For compatibility with the Ringtail application, UI extensions must support all browsers that Ringtail supports&mdash;as of March 2018, this includes Internet Explorer 11, Chrome, and Edge. For more information, see the client computer requirements in the Ringtail Help.

## Build Your Extension
To communicate with Ringtail, initialize the SDK and then hook up listeners for the events that you are interested in. Here's an example that listens for and displays active document changes:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Ringtail UI Extension Test App</title>
    <script src="Ringtail.js" type="text/javascript"></script>
</head>
<body>
    <b>Active Document</b>
    <p>
        <table>
            <tr><td>Document ID</td><td class="active-doc-id"></td></tr>
            <tr><td>Main ID</td><td class="active-main-id"></td></tr>
            <tr><td>Entity Type ID</td><td class="active-entity-type-id"></td></tr>
            <tr><td>Result Set ID</td><td class="active-result-set-id"></td></tr>
        </table>
    </p>

    <script>
        function handleActiveDocumentChanged(message) {
            document.querySelector('.active-doc-id').innerHTML = message.data.documentId;
            document.querySelector('.active-main-id').innerHTML = message.data.mainId;
            document.querySelector('.active-entity-type-id').innerHTML = message.data.entityTypeId;
            document.querySelector('.active-result-set-id').innerHTML = message.data.resultSetId;
        }

        // Establish communication with Ringtail
        Ringtail.initialize().then(function () {
            // Listen for ActiveDocument changes
            Ringtail.on('ActiveDocument', handleActiveDocumentChanged);
        });
    </script>
</body>
</html>
```

## Add Your Extension to Ringtail
Once you are ready to test and deploy your extension, you need to add it to a Ringtail environment. Portal and System Administrators have access to the UI Extensions area of the Ringtail portal and can install and configure extensions.

There are three steps to getting a UI extension to show up in Ringtail.

### 1. Add the UI extension in the portal
In the UI Extensions area of the Ringtail portal you have several options for adding an extension.

  1. For simple UI extensions like the one above, you can specify required settings in the Basic section of the Add dialog.
  1. For more complex UI extensions using custom fields and statistics counters, you need to specify settings via an [extension manifest](ExtensionManifest.md). Once ready, you can import the manifest into the Advanced section of the Add dialog.

### 2. Assign the UI Extension to Organizations and Cases
Once the extension is added, click on it in the UI extension list to assign it to one or more Organizations and Cases. This step grants access to the extension for users in the allowed cases and causes Ringtail to automatically create any fields and statistics counters specified in the extension manifest.

### 3. In each Ringtail case, grant access to the extension for one or more user groups
UI extensions assigned to cases show up in the Features page of the Security area for case administrators. Allow access to the extension for your current user group and refresh to see it show up in Ringtail!

## Security Considerations
During initialization, Ringtail provides each UI extension with authentication credentials, allowing the UI extension to make Ringtail Connect API calls on behalf of the current user. This provides secure access to Ringtail&mdash;but, it does not prove the legitimacy of either the user or the Ringtail instance that hosts the UI extension. Therefore, if your UI extension maintains its own data, you must take further steps to establish trust with Ringtail.

### Security Guidelines

1. __Restrict the domains that are allowed to host your UI extension to only those you expect.__ Pass the allowed domains in an array to `Ringtail.initialize([domain1, domain2, ...])`. This step prevents unknown sites from hosting your UI extension and spoofing Ringtail.
1. __Verify Ringtail authenticity by using an authentication secret.__ When you install the UI extension in Ringtail, provide an authentication secret in the Ringtail UI. This enables Ringtail to construct and sign a [JSON Web Token (JWT)](https://jwt.io/) containing context and configuration data.

#### Full Example
Provide the allowed domain list and validate the provided JWT during initialization of the UI extension:
```js
// Allow communication only with these domains
var domainWhitelist = ['https://ringtail.com', 'https://portal02.ringtail.com'];

Ringtail.initialize(domainWhitelist).then(function () {
    Ringtail.Context.externalAuthToken // <-- Send to your server for validation/login
    // NOTE: This JWT contains everything in Ringtail.Context as additional claims
});
