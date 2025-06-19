# Listener Self Cleanup Event Pattern (LSC)

The **LSC pattern** gives cleanup control to the **listener**. Instead of the notifier removing listeners, it provides a `release` function via `listener.onCleanup`. The listener decides *when* and *how* to unregister.

## Problems with Traditional Cleanup

### Manual Unregistration
- Easy to forget  
- Memory leaks  
- Scattered, repetitive code

### Framework Auto-Cleanup
- Hard to understand  
- Doesn’t play well with other event systems

## Benefits of LSC

- **Clear Intent:** `onCleanup` makes cleanup explicit  
- **Flexible:** Listeners choose *when* to unregister (e.g. timeout, condition)  
- **Minimal Boilerplate:** Keeps logic local to listener  

## Works In
- JavaScript  
- Python  
- Ruby  
- Lua  
(Any language with first-class functions)

## JavaScript Example

```js
class Notifier {
  constructor() {
    this.listeners = new Set();
  }

  addListener(listener) {
    const release = () => this.removeListener(listener);
    if (!this.listeners.has(listener)) {
      this.listeners.add(listener);
      if (typeof listener.onCleanup === 'function') {
        listener.onCleanup(release); // LSC pattern
      }
    }
    return release;
  }

  removeListener(listener) {
    this.listeners.delete(listener);
  }

  notify(...args) {
    this.listeners.forEach(listener => listener(...args));
  }
}

```
```js
// Example listener
function listener(data) {
  console.log('Received:', data);
}
// Optional auto-cleanup: unregister after 5 seconds
listener.onCleanup = (release) => {
  setTimeout(release, 5000);
};
let notifier = new Notifier()
notifier.addListener(listener);
// Later usage
notifier.notify('event fired!');
``` 
## Useful Use Cases for LSC Pattern 
- Complex reactive applications (e.g., UI frameworks)
- Dynamic node graphs (e.g., visual scripting, geometry modeling)
- Event-driven systems with long-lived objects
- Temporary observers (e.g., animation triggers, game logic)
- Resource managers (e.g., network, cache, file watching)
- Logging or debugging hooks with limited lifespan
- Modular plugin systems where listeners self-register

License: MIT Copyright: 2025 Author: [Khánh Nguyễn](https://github.com/huukhanhnguyen)
