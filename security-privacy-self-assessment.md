https://www.w3.org/TR/security-privacy-questionnaire/

### 3.1 Does this specification deal with personally-identifiable information?

No.

### 3.2 Does this specification deal with high-value data?

No.

### 3.3 Does this specification introduce new state for an origin that persists across browsing sessions?

No.

### 3.4 Does this specification expose persistent, cross-origin state to the web?

As not all device types may support a "lock" state, detecting such a state provides some additional distinguishing information about the user's device (i.e. potentially one bit of entropy for fingerprinting), but as this correlates with the native platform this is generally inferrable from information exposed by the user agent header already.

### 3.5 Does this specification expose any other data to an origin that it doesn’t currently have access to?

Yes - the status of a "screen lock" or equivalent is not detectable by script today - other than e.g. [Wake Lock](https://w3c.github.io/wake-lock/) to detect the absence.

Detection of "idle" vs. "active" can be done by script today by watching for UI events, but this is restricted to events within windows/frames controlled by the origin.

The "idle" state can be described as "time since last user interface event" and is global state which, with this API, can be observed cross-origin. In one hypothetical attack, with sub-second detection it would be possible to infer keystrokes and thus guess passwords being entered in other windows. For this reason, idle time detection must be appropriately coarse. In another attack scenario different origins could collaborate to corrolate idle state change events on the back-end, allowing identification of a user session across origins. For this reason, this capability must be limited to a small set of trustworthy origins by limiting it to top-level frames with notification permission. 

### 3.6 Does this specification enable new script execution/loading mechanisms?

No.

### 3.7 Does this specification allow an origin access to a user’s location?

No.

### 3.8 Does this specification allow an origin access to sensors on a user’s device?

Not directly. A device may use sensors to control lock state (e.g. proximity sensor to lock, fingerprint sensor to unlock) but the sensor data itself is not exposed.

### 3.9 Does this specification allow an origin access to aspects of a user’s local computing environment?

See the answer to 3.5 above. The user's idle state is an aspect of their local computing environment which extends beyond the boundaries of a single site.

### 3.10 Does this specification allow an origin access to other devices?

No.

### 3.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

No.

### 3.12 Does this specification expose temporary identifiers to the web?

No.

### 3.13 Does this specification distinguish between behavior in first-party and third-party contexts?

This capability is not allowed in third-party contexts.

### 3.14 How should this specification work in the context of a user agent’s "incognito" mode?

This capability should be disabled in "incognito" modes to avoid exposing the same global state to two content inside and outside of "incognito" mode simultaneously.

### 3.15 Does this specification persist data to a user’s local device?

No.

### 3.16 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

_Not yet!_

**TODO: Make sure this is included when the spec is written.**

### 3.17 Does this specification allow downgrading default security characteristics?

No.
