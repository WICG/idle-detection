# idle-detection

A proposal for an idle detection and notification API for the web

* This _is_ about detecting when the user is away from the keyboard or the screen is locked.
* This is _not_ about asynchronously scheduling work when the system is idle. See [requestIdleCallback](https://www.w3.org/TR/requestidlecallback/) for that one.

## Taste of the API

Modeled on Chrome's [chrome.idle](https://developer.chrome.com/apps/idle) API, the API could be used in the following way:

```js
// TODO: Examples of explicit permission request.

navigator.idle.setDetectionInterval(2 * 60); // Only check every 2 minutes. Default is 1 minute.

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

## Permissions

A new [permission](https://w3c.github.io/permissions/) would be associated with this functionality. The permission might be auto-granted based on heuristics, such as user engagement, having "installed" the web site as a bookmark or desktop/homescreen icon, or having granted similar permissions such as [Wake Lock](https://w3c.github.io/wake-lock/).

Using the `setDetectionInterval()` or `query()` API will trigger a permission request (if not already granted/blocked).

> TODO: What's the best model to follow these days?

## Security and Privacy

> TODO: Fill out [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

* There are definitely privacy implications here, mandating a new permission.
* There is a new way of causing work to be scheduled, but no new network or storage functionality is offered.

## Prior Work

* Chrome's [chrome.idle](https://developer.chrome.com/apps/idle) API for apps/extensions, which is a direct inspiration for this work.
  * Also exposed to Extensions in Firefox [MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/idle)
* [Idle.js](http://shawnmclean.com/detecting-if-user-is-idle-away-or-back-by-using-idle-js/), a JavaScript library for detecting user idle/active behavior in a web page.
