# idle-detection

A proposal for an idle detection and notification API for the web

* This _is_ about detecting when the user is away from the keyboard or the screen is locked.
* This is _not_ about asynchronously scheduling work when the system is idle. See [requestIdleCallback](https://www.w3.org/TR/requestidlecallback/) for that one.

The API should provide a means to _detect_ the user's idle status (active, idle, locked), and a power-efficient way to be _notified_ of changes to the status without polling from script.

## Use cases

* Chat application: presenting a user's status to other users
* Showing timely alerts - e.g. deferring displaying feedback until the user returns to an active state
* Automatically pausing media when the screen is locked

### Why is a built-in API better than JS?

* JS can only detect active/idle within a page by watching for UI events; the user agent can observe any interaction with the browser (or query the OS) to give a more accurate reflection of the state
* Screen lock detection
* Exposure in Workers w/o proxying from a window
* Avoiding costly polling from script

## Model

The API assumes that there is some level of engagement between the user, user agent, and operating system of the device in use. This is represented in three states:

* **active** - the user is interacting with the user agent
* **idle** - the user has not interacted with the user agent for some period of time
* **locked** - the system has an active screen lock preventing interaction with the user agent

Distinguishing "active" from "idle" requires heuristics that may differ across user, user agent, and operating system. It should also be a reasonably coarse threshold. (See Privacy)

The model intentionally does not formally distinguish between interaction with particular content (i.e. the web page in a tab using the API), the user agent as a whole, or the operating system; this definition is left to the user agent.

> Example: The user is interacting with an operating system providing multiple virtual desktops. The user may be actively interacting with one virtual desktop, but unable to see the content of another virtual desktop. A user agent presenting content on the second virtual desktop may report an "idle" state rather than an "active" state.

## Taste of the API

Modeled on Chrome's [chrome.idle](https://developer.chrome.com/apps/idle) API, the API could be used in the following way:

```js
// TODO: Examples of explicit permission request.

// Define "idle" as two minutes of inactivity.
navigator.idle.setDetectionThreshold(2 * 60); 

// Initialize the UI with the current state.
navigator.idle.query().then(state => {
  update_user_state(state);
})

// Watch for future state changes.
navigator.idle.addEventListener('changed', event => {
  update_user_state(event.state);
});

// Idle state will be 'active', 'idle', or 'locked'.
function update_user_state(state) {
  switch (state) {
  case 'active':
    document.body.style.backgroundColor = 'green';
    break;
  case 'idle':
    document.body.style.backgroundColor = 'yellow';
    break;
  case 'locked':
    document.body.style.backgroundColor = 'red';
    break;
  }
}
```

> Issue: This API sketch makes the detection threshold global for an execution context. This means two libraries running in the same window/worker would fight over the threshold.

## Device Capabilities

Not all devices implement a notion equivalent to "locking". Mobile devices typically have a screen lock, and desktop systems have screen savers that may or may not actually require a password to unlock. User agents can employ a heuristic to define "locked", such as any state where the user cannot observe the application state without first taking action.

## Permissions

A new [permission](https://w3c.github.io/permissions/) would be associated with this functionality. The permission might be auto-granted based on heuristics, such as user engagement, having "installed" the web site as a bookmark or desktop/homescreen icon, or having granted similar permissions such as [Wake Lock](https://w3c.github.io/wake-lock/).

Using the `setDetectionThreshold()` or `query()` API will trigger a permission request (if not already granted/blocked).

> TODO: What's the best model to follow these days?

## Security and Privacy

> TODO: Fill out [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

* There are definitely privacy implications here, mandating a new permission.
* There is a new way of causing work to be scheduled, but no new network or storage functionality is offered.
* The threshold to distinguish between "active" and "idle" must be coarse enough to preclude inferring too much about user activity 
    * At an extreme, typing cadence can be used to guess passwords.
    * Users with physical or cognitive impairments may require more time to interact with user agents and content. The API should not allow distinguishing such users, or limiting their ability to interact with content.

## Prior Work

* Chrome's [chrome.idle](https://developer.chrome.com/apps/idle) API for apps/extensions, which is a direct inspiration for this work.
  * Also exposed to Extensions in Firefox [MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle)
* Attempts to do this from JS running on the page:
  * [idle.ts](https://github.com/dropbox/idle.ts) from Dropbox
  * [Idle.js](http://shawnmclean.com/detecting-if-user-is-idle-away-or-back-by-using-idle-js/)
