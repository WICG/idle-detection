<img src="https://raw.githubusercontent.com/inexorabletash/idle-detection/master/logo-idle.png" height="100" align=right>

# User Idle Detection

This is a proposal to give developers the ability to be notified when users go idle (e.g. they don’t interact with the keyboard/mouse/screen, when a screensaver kicks in and/or when the screen gets locked) past a certain time limit, even beyond their content area (e.g. when users move to a different window/tab).

Native applications / extensions (e.g. [Chrome apps](https://developer.chrome.com/apps/idle), [Android apps](https://stackoverflow.com/questions/8317331/detecting-when-screen-is-locked), [Firefox extensions](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle), [Edge extensions](https://github.com/MicrosoftDocs/edge-developer/blob/master/microsoft-edge/extensions/api-support/supported-apis.md#idle)) use idle detection to notify other users (e.g. chat apps letting other users know that the user isn’t active), to show timely alerts (e.g. "welcome back" when a user goes idle and restarts their task) or to pause media (e.g. to save bandwidth when the user is idle).

The API should provide a means to _detect_ the user's idle status (active, idle, locked), and a power-efficient way to be _notified_ of changes to the status without polling from script.

Feedback: [WICG Discourse Thread](https://discourse.wicg.io/t/idle-detection-api/2959) &mdash; [Issues](https://github.com/inexorabletash/idle-detection/issues)

## Use cases

* Chat application: presenting a user's status to other users
* Showing timely notifications - e.g. deferring displaying feedback until the user returns to an active state

## Relationship with other APIs

* As opposed to the [requestIdleCallback](https://www.w3.org/TR/requestidlecallback/), this is _not_ about asynchronously scheduling work when the **system** is idle.
* As opposed to the [Page Visibility API](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API), this API enables detecting idleness even after a page is no longer visible (e.g. after the page is no longer visible, is the user still around? if i showed a notification, would it be perceived?).

## Polyfills

Currently, web apps (e.g. Dropbox’s [idle.ts](https://github.com/dropbox/idle.ts)) are constrained to their own content area:

1. costly polling for input events or 
1. listening to [visibility changes](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API)

Script can't tell today when a user goes idle outside of its content area (e.g. whether a user is on a different tab or logged out of the computer altogether).

## Model

The API assumes that there is some level of engagement between the user, user agent, and operating system of the device in use. This is represented in two dimensions:

1. The user idle state
  * **active**/**idle** - the user has / has not interacted with the user agent for some period of time
2. The screen idle state
  * **locked**/**unlocked** - the system has an active screen lock preventing interaction with the user agent

Distinguishing "active" from "idle" requires heuristics that may differ across user, user agent, and operating system. It should also be a reasonably coarse threshold (See Privacy).

The model intentionally does not formally distinguish between interaction with particular content (i.e. the web page in a tab using the API), the user agent as a whole, or the operating system; this definition is left to the user agent.

> Example: The user is interacting with an operating system providing multiple virtual desktops. The user may be actively interacting with one virtual desktop, but unable to see the content of another virtual desktop. A user agent presenting content on the second virtual desktop may report an "idle" state rather than an "active" state.

## API Design

There are multiple alternatives to be considered here. Here are the ones that we ran into:

* [IdleObserver](#IdleObserver)
* [navigator.idle.query()](#navigatoridlequery) and variations ([chrome.idle](#chrome.idle.query), [browser.idle](#browser.idle.query))

Here are some guidance on [Events vs Observers](https://w3ctag.github.io/design-principles/#events-vs-observers) we got from the TAG review.

### Alternatives Considered

#### IdleDetector

This formulation is inspired by [@kenchris's feedback](https://github.com/w3ctag/design-reviews/issues/336#issuecomment-470077151), the overall guidance on [Observers vs EventTargets](https://w3ctag.github.io/design-principles/#events-vs-observers), and the [Sensor API], specifically, the `Accelerometer` class.

```
let idleDetector = new IdleDetector({ threshold: 60 });
idleDetector.addEventListener('reading', () => reloadOnShake(idleDetector));
idleDetector.start();
```

And

```
const reading = await IdleDetector.read({ threshold: 2 * 60 });
```

#### IdleObserver

This formulation is inspired by the [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver), the [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) and the [PerformanceObserver](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) APIs.

```js
const observer = new IdleObserver({user, screen} => {
  // do stuff
});

// Define "idle" as two minutes of inactivity.
observer.observe({threshold: 2*60});
```

Open questions:

* Should we allow observer.disconnect()?
* Should we allow multiple new IdleObserver() to run concurrently?


#### navigator.idle.query

This formulation is closer to chrome's [`chrome.idle.query()`](https://developer.chrome.com/apps/idle) API and [`browser.idle,queryState()`](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle/queryState):

```js
const monitor = navigator.idle.query({threshold: 2 * 60});

// Listen to state changes
monitor.addEventListener('change', (state) => {
  // do stuff
});
```

Or, if you only care about the current state:

```js
navigator.idle.query({threshold: 2 * 60})
  .addEventListener({once: true}, (state) => {
    // do stuff
  });
```

Open questions:

* do we have a preference between `navigator.idle.query()` or `navigator.idle.observe()`?
* should we `navigator.idle.query()` return a `Promise` such that the current state is returned easily?

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

## Platforms

All supported platforms (linux, mac, windows, android, ios) have some notion of:

- screen off / saver
- screen locked

### desktop

On desktop (linux, mac, windows), a screen saver (from the time monitors were damaged when the same pixels were displayed) kicks in after a certain period of inactivity. If set up, the screen also gets locked after the user goes inactive for more time. Both of these events are observable by engines.

### mobile

On mobile (android), the screen gets dimmed a few moments after the user goes inactive (to save battery, not pixels) but isn't observable by engines (on android). The screen gets eventually turned off (to save further battery) if the user remains inactive for a configurable amount of time (typically 30 seconds), and that's observable by engines. When the screen goes off, the screen also typically gets locked (unlocked by Swipe, Pattern, PIN or Password), although it can be configured to be left off but unlocked.

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

At least initially, per [TAG review](https://github.com/w3ctag/design-reviews/issues/336#issuecomment-460482399), we don't see any major gains we would get allowing the API to be called outside of top-level frames, so restricting it seems like a good starting point (and/or, perhaps, delegation via Feature Policy or a `sandbox` attribute).

## Prior Work

* Chrome's [chrome.idle](https://developer.chrome.com/apps/idle) API for apps/extensions, which is a direct inspiration for this proposal.
  * Also exposed to Extensions in Firefox [MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle)
  * And [Edge](https://github.com/MicrosoftDocs/edge-developer/blob/master/microsoft-edge/extensions/api-support/supported-apis.md#idle)
  * That API has a global (per-execution-context) threshold and one source of events. This makes it difficult for two components on the same page to implement different thresholds.
* Attempts to do this from JS running on the page:
  * [idle.ts](https://github.com/dropbox/idle.ts) from Dropbox
  * [Idle.js](http://shawnmclean.com/detecting-if-user-is-idle-away-or-back-by-using-idle-js/)
