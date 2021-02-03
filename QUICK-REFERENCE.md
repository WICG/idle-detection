---
recipe: api-interface
title: 'IdleDetector'
mdn_url: /en-US/docs/Web/API/IdleDetector
specifications: https://wicg.github.io/idle-detection/#api-idledetector
browser_compatibility: api.IdleDetector
---

**When this feature ships, the content below will live on MDN under
[developer.mozilla.org/en-US/docs/Web/API](https://developer.mozilla.org/en-US/docs/Web/API).**

## Description

The `IdleDetector` interface of the Idle Detection API provides events
indicating when the user is no longer interacting with their device or the
screen has locked.

This interface requires a secure context.

## Constructor

Creates a new `IdleDetector` object.

## Properties

**`IdleDetector.userState`**

Returns either `"active"` to indicate that the user has interacted with the
device within the threshold provided to `start()` or `"idle"` if they have not.
This attribute returns `null` before `start()` is called.

**`IdleDetector.screenState`**

Returns either `"locked"` if the device's screen is locked or `"unlocked"` if it
is not. This attribute returns `null` before `start()` is called.

## Events

**`IdleDetector.onchange`**

Called when the value of `userState` or `screenState` has changed. This method
receives an `Event` object.

## Methods

**`IdleDetector.requestPermission()`**

Returns a `Promise` that resolves when the user has chosen whether or not to
grant the origin access to their idle state. Resolves with `"granted"` on
acceptance and `"denied"` on refusal.

**`IdleDetector.start()`**

Returns a `Promise` that resolves when the detector has started listening for
changes in the user's idle state. `userState` and `screenState` are populated
with their initial values.

## Examples

The following example shows creating a detector and logging changes to the
user's idle state. A button is used to get the necessary user activation before
requesting permission.

```js
button.addEventListener('click', async () => {
  if (await IdleDetector.requestPermission() != "granted") {
    console.error("Idle detection permission denied.");
    return;
  }

  const idleDetector = new IdleDetector();
  idleDetector.addEventListener('change', () => {
    console.log(`Idle change: ${idleDetector.userState}, ${idleDetector.screenState}.`);
  });    
  await idleDetector.start({ threshold: 60000 });
});
```
