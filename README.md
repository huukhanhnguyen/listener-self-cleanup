# Subscriber Controled Cleanup Pattern (SCC)

The **SCC pattern** gives cleanup control to the **Subscriber**. Instead of the notifier removing listeners, it provides a `release` function via `listener.onCleanup`. The subscriber decides *when* and *how* to unregister.
## Terminology Clarification
To avoid confusion, we define the terms used in this pattern:
- **Emitter**, **Publisher**, and **Notifier** refer to the **subject**, which is the source of events or signals. It maintains a list of registered listeners.
- **Subscriber**, **Listener**, **Observer**, and **Callback** refer to the **receiving function or entity** that reacts when notified.
This pattern applies regardless of which terminology your framework uses.
## Problems with Traditional Cleanup

### Manual Unregistration
- Easy to forget  
- Memory leaks  
- Scattered, repetitive code

### Framework Auto-Cleanup
- Hard to understand  
- Doesn’t play well with other event systems

## Works In
- JavaScript  
- Python  
- Ruby  
- Lua  
(Any language with first-class functions)

## JavaScript Example
### Basic Implementation
This example demonstrates the core idea of the Subscriber Self Control Pattern (SCC), where the subscriber controls cleanup via `onCleanup`:
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
        listener.onCleanup(release); // Subscriber Self Control Pattern
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
### Production-Ready Implementation
For production environments, the Notifier should support multiple event types, include error handling, and provide additional cleanup options. Below is a robust implementation:
```javascript
class Notifier {
    constructor() {
        this.listeners = {};
    }
    addListener(event, listener) {
        if (typeof event !== 'string' || typeof listener !== 'function') {
            throw new Error('Event name must be a string, listener must be a function');
        }
        this.listeners[event] = this.listeners[event] || new Set();
        const cleanup = () => this.removeListener(event, listener);
        if (!this.listeners[event].has(listener)) {
            this.listeners[event].add(listener);
            if (typeof listener.onCleanup === 'function') {
                listener.onCleanup(cleanup); // Subscriber Self Control Pattern
            }
        }
        return cleanup;
    }

    removeListener(event, listener) {
        if (typeof event !== 'string' || typeof listener !== 'function') {
            throw new Error('Event name must be a string, listener must be a function');
        }
        if (this.listeners[event] && this.listeners[event].has(listener)) {
            this.listeners[event].delete(listener);
            if (this.listeners[event].size === 0) {
                delete this.listeners[event];
            }
        }
    }

    notify(event, ...args) {
        if (this.listeners[event]) {
            const listeners = [...this.listeners[event]];
            listeners.forEach(listener => {
                try {
                    listener(...args);
                } catch (error) {
                    console.error(`Error in listener for event ${event}:`, error);
                }
            });
        }
    }

    removeAllListeners(event) {
        if (event) {
            if (this.listeners[event]) {
                delete this.listeners[event];
            }
        } else {
            this.listeners = {};
        }
    }
}
```
```
function listener(data) {
  console.log('Received:', data);
}
listener.onCleanup = (release) => {
  setTimeout(release, 5000); // Unsubscribe after 5 seconds
  window.addEventListener('click', release, { once: true }); // Or on click
};
const notifier = new Notifier();
notifier.addListener('userLogin', listener);
notifier.notify('userLogin', 'User logged in!');
```
## Benefits
- **Programmable Unsubscription**: The `listener.onCleanup` function allows subscribers to define custom unsubscription logic, triggered by registered events (e.g., timeouts, user actions, or specific conditions), ensuring reliable cleanup without manual intervention.
- **Clear Intent:** `onCleanup` makes cleanup explicit
- **Flexible:** Listeners choose *when* to unregister (e.g., timeout, condition)
- **Minimal Boilerplate:** Keeps logic local to listener
  ## Extending SCC for State Management and Reactivity
The Subscriber Self Control Pattern (SCC) is well-suited for creating state-driven objects, such as those used in reactive programming or UI frameworks. By extending the `Notifier` class, you can build objects that manage state changes and notify listeners efficiently, while allowing subscribers to control their own cleanup. This reduces memory leaks and simplifies state management in dynamic applications.

- **State-Driven Objects**: SCC enables listeners to react to state changes (e.g., via a "change" event) and unsubscribe when no longer needed.
- **Reactive Applications**: Perfect for scenarios where state updates trigger UI updates or other side effects.
- **Clean Resource Management**: Subscribers define their own cleanup logic, ensuring resources are released appropriately.

### Example: State Management
Below is an example of a `State` class that extends `Notifier` to manage state changes:
```javascript
class State extends Notifier {
    constructor(value) {
        super(); // Initialize Notifier's listeners
        this.value = value;
    }

    setValue(newValue) {
        if (this.value !== newValue) { // Only notify on actual changes
            this.value = newValue;
            this.notify('change', newValue); // Notify listeners of state change
        }
    }

    getValue() {
        return this.value; // Getter for accessing current state
    }
}
```
```
const state = new State(0); // Initial state: 0

function listener(newValue) {
    console.log('State changed to:', newValue);
}
listener.onCleanup = (release) => {
    setTimeout(release, 5000); // Unsubscribe after 5 seconds
    window.addEventListener('click', release, { once: true }); // Or on user click
};

state.addListener('change', listener);
state.setValue(1); // Logs: State changed to: 1
state.setValue(2); // Logs: State changed to: 2
```
## Useful Use Cases 
- Complex reactive applications (e.g., UI frameworks)
- Dynamic node graphs (e.g., visual scripting, geometry modeling)
- Event-driven systems with long-lived objects
- Temporary observers (e.g., animation triggers, game logic)
- Resource managers (e.g., network, cache, file watching)
- Logging or debugging hooks with limited lifespan
- Modular plugin systems where listeners self-register
License: MIT Copyright: 2025 Author: [Khánh Nguyễn](https://github.com/huukhanhnguyen)
