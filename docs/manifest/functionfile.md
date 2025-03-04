---
title: FunctionFile element in the manifest file
description: Specifies the source code file for operations that an add-in exposes through add-in commands that execute a JavaScript function instead of displaying UI.
ms.date: 05/09/2022
ms.localizationpriority: medium
---

# FunctionFile element

Specifies the source code file for operations that an add-in exposes in one of the following ways.

* Add-in commands that execute a JavaScript function instead of displaying UI.
* Keyboard shortcuts that execute a JavaScript function.

**Add-in type:** Task pane, Mail

**Valid only in these VersionOverrides schemas**:

- Task pane 1.0
- Mail 1.0
- Mail 1.1

For more information, see [Version overrides in the manifest](/office/dev/add-ins/develop/add-in-manifests#version-overrides-in-the-manifest).

The **FunctionFile** element is a child element of [DesktopFormFactor](desktopformfactor.md) or [MobileFormFactor](mobileformfactor.md). The `resid` attribute of the **FunctionFile** element can be no more than 32 characters and is set to the value of the `id` attribute of a **Url** element in the **Resources** element that contains the URL to an HTML file that contains or loads all the JavaScript functions used by UI-less add-in command buttons, as defined by the [Control element](control.md).

> [!NOTE]
> When the add-in is configured to use a [shared runtime](/office/dev/add-ins/develop/configure-your-add-in-to-use-a-shared-runtime), the functions in the code file run in the same JavaScript runtime (and share a common global namespace) as the JavaScript in the add-in's task pane (if any).
>
> The **FunctionFile** element and the associated code file also have a special role to play with [custom keyboard shortcuts](/office/dev/add-ins/design/keyboard-shortcuts), which require a shared runtime.

The following is an example of the **FunctionFile** element.

```XML
<DesktopFormFactor>
  <FunctionFile resid="residDesktopFuncUrl" />
  <ExtensionPoint xsi:type="PrimaryCommandSurface">
    <!-- information about this extension point -->
  </ExtensionPoint>

  <!-- You can define more than one ExtensionPoint element as needed -->

</DesktopFormFactor>
```

The JavaScript in the HTML file indicated by the **FunctionFile** element must call `Office.initialize` and define named functions that take a single parameter: `event`. The functions should use the `item.notificationMessages` API to indicate progress, success, or failure to the user. It should also call `event.completed` when it has finished execution. The name of the functions are used in the [FunctionName](action.md#functionname) element for UI-less buttons.

The following is an example of the contents of a `<script>` tag in an HTML file. The code defines and registers a `trackMessage` function.

```js
Office.initialize = function () {
    // Your add-in's initialization logic, if any, goes here.
}

function trackMessage (event) {
    var buttonId = event.source.id;    
    var itemId = Office.context.mailbox.item.id;
    // save this message
    event.completed();
}

// Register the function with Office.
Office.actions.associate("trackMessage", trackMessage);
```

You can also define and register the function specified by the **FunctionName** element in a separate JavaScript file that is loaded by the HTML file. The following is an example of such a file.

```js
// The initialize function must be assigned each time a new page is loaded.
Office.initialize = function (reason) {
    // If you need to initialize something you can do so here.
};


// Define the function.
function writeText(event) {

    // Implement your custom code here. The following code is a simple example.

    Office.context.document.setSelectedDataAsync("ExecuteFunction works. Button ID=" + event.source.id,
        function (asyncResult) {
            var error = asyncResult.error;
            if (asyncResult.status === "failed") {
                // Show error message.
            }
            else {
                // Show success message.
            }
        });
    // Calling event.completed is required. event.completed lets the platform know that processing has completed.
    event.completed();
}

// Register the function with Office.
Office.actions.associate("writeText", writeText);
```

> [!IMPORTANT]
> The call to `event.completed` signals that you have successfully handled the event. When a function is called multiple times, such as multiple clicks on the same add-in command, all events are automatically queued. The first event runs automatically, while the other events remain on the queue. When your function calls `event.completed`, the next queued call to that function runs. You must call `event.completed`; otherwise your function will not run.
