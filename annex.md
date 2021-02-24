There are multiple alternatives to be considered here. Here are the ones that we ran into:

* [IdleObserver](#IdleObserver)
* [navigator.idle.query()](#navigatoridlequery) and variations ([chrome.idle](#chrome.idle.query), [browser.idle](#browser.idle.query))

Here are some guidance on [Events vs Observers](https://w3ctag.github.io/design-principles/#events-vs-observers) we got from the TAG review.

### Alternatives Considered

#### IdleDetector

This formulation is inspired by [@kenchris's feedback](https://github.com/w3ctag/design-reviews/issues/336#issuecomment-470077151), the overall guidance on [Observers vs EventTargets](https://w3ctag.github.io/design-principles/#events-vs-observers), and the [Sensor API](https://w3c.github.io/sensors/#feature-detection), specifically, the [`Accelerometer`](https://w3c.github.io/sensors/#feature-detection) class.

```js
async function main() {
  // feature detection.
  if (!window.IdleDetector) {
    console.log("IdleDetector is not available :(");
    return;
  }
  
  console.log("IdleDetector is available! Go idle!");
  
  try {
    let idleDetector = new IdleDetector({ threshold: 60 });
    idleDetector.addEventListener('change', ({user, screen}) => { 
      console.log(`idle change: ${user}, ${screen}`);
    });
    await idleDetector.start();
  } catch (e) {
    // deal with initialization errors.
    // permission denied, running outside of top-level frame, etc
  }
};
```

And for a one-shot reading of the state:

```js
const {user, screen} = await IdleDetector.read({ threshold: 2 * 60 });
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
monitor.addEventListener('change', ({user, screen}) => {
  // do stuff
});
```

Or, if you only care about the current state:

```js
navigator.idle.query({threshold: 2 * 60})
  .addEventListener({once: true}, ({user, screen}) => {
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
