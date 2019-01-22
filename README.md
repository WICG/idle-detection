<img src="https://raw.githubusercontent.com/inexorabletash/idle-detection/master/logo-idle.png" height="100" align=right>

# User Idle Detection

This is a proposal to give developers the ability to be notified when users go idle (e.g. they don’t interact with the keyboard/mouse/screen, when a screensaver kicks in and/or when the screen gets locked) past a certain time limit, even beyond their content area (e.g. when users move to a different window/tab).

Native applications / extensions (e.g. [Chrome apps](https://developer.chrome.com/apps/idle), [Android apps](https://stackoverflow.com/questions/8317331/detecting-when-screen-is-locked), [Firefox extensions](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle), [Edge extensions](https://github.com/MicrosoftDocs/edge-developer/blob/master/microsoft-edge/extensions/api-support/supported-apis.md#idle)) use idle detection to notify other users (e.g. chat apps letting other users know that the user isn’t active), to show timely alerts (e.g. "welcome back" when a user goes idle and restarts their task) or to pause media (e.g. to save bandwidth when the user is idle).

This is _not_ to be confused about asynchronously scheduling work when the **system** is idle (see [requestIdleCallback](https://www.w3.org/TR/requestidlecallback/)).

The API should provide a means to _detect_ the user's idle status (active, idle, locked), and a power-efficient way to be _notified_ of changes to the status without polling from script.

Feedback: [WICG Discourse Thread](https://discourse.wicg.io/t/idle-detection-api/2959) &mdash; [Issues](https://github.com/inexorabletash/idle-detection/issues)

## Use cases

* Chat application: presenting a user's status to other users
* Showing timely alerts - e.g. deferring displaying feedback until the user returns to an active state

### Why is a built-in API better than tracking input events (etc) in JS?

Currently, web apps (e.g. Dropbox’s [idle.ts](https://github.com/dropbox/idle.ts)) are constrained to their own content area:

1. costly polling for input events or 
1. listening to [visibility changes](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API)

Either way, script can't tell today when a user goes idle outside of its content area (e.g. whether a user is on a different tab or logged out of the computer altogether).

## Model

The API assumes that there is some level of engagement between the user, user agent, and operating system of the device in use. This is represented in three states:

* **active** - the user is interacting with the user agent
* **idle** - the user has not interacted with the user agent for some period of time
* **locked** - the system has an active screen lock preventing interaction with the user agent

Distinguishing "active" from "idle" requires heuristics that may differ across user, user agent, and operating system. It should also be a reasonably coarse threshold. (See Privacy)

The model intentionally does not formally distinguish between interaction with particular content (i.e. the web page in a tab using the API), the user agent as a whole, or the operating system; this definition is left to the user agent.

> Example: The user is interacting with an operating system providing multiple virtual desktops. The user may be actively interacting with one virtual desktop, but unable to see the content of another virtual desktop. A user agent presenting content on the second virtual desktop may report an "idle" state rather than an "active" state.

## Taste of the API

There are multiple alternatives to be considered here. The following is an API inspired by the [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) and the [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) APIs.


```js
const observer = new IdleObserver({state} => {
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
});

// Define "idle" as two minutes of inactivity.
observer.observe({threshold: 2*60});
```

### Alternatives Considered

#### [chrome.idle](https://developer.chrome.com/apps/idle)

Modeled roughly on Chrome's [chrome.idle](https://developer.chrome.com/apps/idle) API, with inspiration from the [Permissions API](https://w3c.github.io/permissions/#permissions-interface), the API could be used in the following way:

```js
async function start_observing_idle() {
  // Define "idle" as two minutes of inactivity.
  // A permission prompt could be shown here, depending on the UA.
  const status = await navigator.idle.query({threshold: 2*60});

  // Use the current status.
  update_user_state(status.state);

  // Respond to future status changes.
  status.addEventListener('change', e => {
    update_user_state(status.state);
  });
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

#### [browser.idle](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle/queryState)

```js
// Has to be polled for changes as opposed to notified (e.g. it is not
// an event target).
browser.idle.queryState( detectionIntervalInSeconds // integer )
```

#### Variations

Here are some variations of the Observer pattern that we could try too:

```js
navigator.idle.observe({threshold: 2*60}, (e) => console.log(e))

let observer = navigator.idle.query({threshold: 2*60}, e => console.log(e))
observe.observe()

// More consistent with MutationObserver, less consistent with
// the other static navigator.* APIs.
let observer = new IdleObserver(e => console.log(e));
observer.observe({threshold: 2*60});
```

## Device Capabilities

Not all devices implement a notion equivalent to "locking". Mobile devices typically have a screen lock, and desktop systems have screen savers that may or may not actually require a password to unlock. User agents can employ a heuristic to define "locked", such as any state where the user cannot observe the application state without first taking action.

## Permissions

A new [permission](https://w3c.github.io/permissions/) would be associated with this functionality. A new [permission name](https://w3c.github.io/permissions/#permission-registry) such as `"idle-detection"` would be registered. The permission might be auto-granted based on heuristics, such as user engagement, having "installed" the web site as a bookmark or desktop/homescreen icon, or having granted similar permissions such as [Wake Lock](https://w3c.github.io/wake-lock/).

Using the `query()` API will trigger a permission request (if not already granted/blocked).

## Security and Privacy

See answers to [Self-Review Questionnaire: Security and Privacy](security-privacy-self-assessment.md)

* There are definitely privacy implications here, mandating a new permission.
* There is a new way of causing work to be scheduled, but no new network or storage functionality is offered.
* The threshold to distinguish between "active" and "idle" must be coarse enough to preclude inferring too much about user activity
    * At an extreme, typing cadence can be used to guess passwords.
    * Users with physical or cognitive impairments may require more time to interact with user agents and content. The API should not allow distinguishing such users, or limiting their ability to interact with content any more than existing observation of UI events.

An implication here is that if implementations clamp the detection threshold, they should also clamp how quickly responses to `query()` are delivered and/or ensure that the responses to `query()` are cached or otherwise provide some granularity that rapid polling with JS does not bypass the clamp.

## Prior Work

* Chrome's [chrome.idle](https://developer.chrome.com/apps/idle) API for apps/extensions, which is a direct inspiration for this proposal.
  * Also exposed to Extensions in Firefox [MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle)
  * And [Edge](https://github.com/MicrosoftDocs/edge-developer/blob/master/microsoft-edge/extensions/api-support/supported-apis.md#idle)
  * That API has a global (per-execution-context) threshold and one source of events. This makes it difficult for two components on the same page to implement different thresholds.
* Attempts to do this from JS running on the page:
  * [idle.ts](https://github.com/dropbox/idle.ts) from Dropbox
  * [Idle.js](http://shawnmclean.com/detecting-if-user-is-idle-away-or-back-by-using-idle-js/)
