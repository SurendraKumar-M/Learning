# React Native / React Native Expo — 5+ Years Interview Preparation

> Comprehensive interview Q&A covering JavaScript, React, React Native, Expo, navigation, state management, networking, storage, performance, native development, testing, architecture, security, production, and scenario-based questions.
>
> **Target:** React Native / React Native Expo interviews at 5+ years level.
>
> **Recommended answer pattern:** Direct answer → internal behavior → practical usage → trade-off → failure mode / follow-up.

---

## Table of Contents

1. [How to Use This Guide](#how-to-use-this-guide)
2. [JavaScript Fundamentals](#part-1--javascript-fundamentals)
3. [React Fundamentals](#part-2--react-fundamentals)
4. [React Native Fundamentals](#part-3--react-native-fundamentals)
5. [Navigation](#part-4--navigation)
6. [Expo and EAS](#part-5--expo-and-eas)
7. [State Management](#part-6--state-management)
8. [Networking and API Architecture](#part-7--networking-and-api-architecture)
9. [Storage and Offline Data](#part-8--storage-and-offline-data)
10. [Performance](#part-9--performance)
11. [Android, iOS, and Native Integration](#part-10--android-ios-and-native-integration)
12. [Testing](#part-11--testing)
13. [Architecture and Code Organization](#part-12--architecture-and-code-organization)
14. [Authentication and Security](#part-13--authentication-and-security)
15. [Production, Release, CI/CD, and OTA](#part-14--production-release-cicd-and-ota)
16. [Scenario-Based 5+ Years Questions](#part-15--scenario-based-5-years-questions)
17. [Coding Questions](#part-16--coding-questions)
18. [Rapid-Fire Questions](#part-17--rapid-fire-questions)
19. [Senior-Level Discussion Questions](#part-18--senior-level-discussion-questions)
20. [Final Revision Checklist](#final-revision-checklist)
21. [Current Official References](#current-official-references)

---

# How to Use This Guide

A 5+ years interviewer is unlikely to judge you only on whether you can define a hook or a component. The panel usually looks for four additional things:

- Whether you understand **why** a technology behaves the way it does.
- Whether you can discuss **trade-offs** instead of presenting one tool as universally correct.
- Whether you can **debug production problems systematically**.
- Whether you can make reasonable **architecture decisions** for a growing application.

For most questions, use this answer format:

> **1. Definition:** What is it?
>
> **2. Internals:** How does it work?
>
> **3. Practical usage:** Where would you use it in a real project?
>
> **4. Trade-off:** What are its limitations?
>
> **5. Scenario:** What can go wrong and how would you debug it?

Do not memorize the exact wording. Learn the underlying model and then explain it naturally.

---

# Part 1 — JavaScript Fundamentals

## Q1. What is the difference between `var`, `let`, and `const`?

### Interview answer

`var` is function-scoped, while `let` and `const` are block-scoped. `var` declarations are hoisted and initialized with `undefined`. `let` and `const` are also hoisted in the language's execution model, but they remain uninitialized and are inaccessible during the Temporal Dead Zone. `const` prevents reassignment of the binding, but it does not make referenced objects immutable.

```js
var a = 10;
let b = 20;
const c = 30;
```

```js
{
  var x = 10;
  let y = 20;
}

console.log(x); // 10
console.log(y); // ReferenceError
```

### Important follow-up

```js
const user = {
  name: 'Surendra',
};

user.name = 'Kumar'; // allowed

// user = {}; // TypeError
```

The binding is constant; the object it references is not automatically frozen.

---

## Q2. What is hoisting?

### Interview answer

Hoisting is the behavior produced by JavaScript's declaration processing before code execution within a scope. Function declarations can be called before their source position. `var` declarations are initialized to `undefined`. `let` and `const` declarations exist in the scope but cannot be accessed before initialization because of the Temporal Dead Zone.

```js
console.log(a);
var a = 10;
// undefined
```

```js
sayHello();

function sayHello() {
  console.log('Hello');
}
```

### Senior follow-up

Do not say that `let` and `const` are “not hoisted.” A better explanation is:

> They are hoisted in the sense that their bindings are created during scope setup, but they are not initialized before execution reaches their declaration.

---

## Q3. What is the Temporal Dead Zone (TDZ)?

### Interview answer

The Temporal Dead Zone is the time between entering a scope and the point where a `let`, `const`, or `class` declaration is initialized. Accessing the binding during that interval throws a `ReferenceError`.

```js
console.log(name); // ReferenceError
let name = 'Surendra';
```

The TDZ helps catch accidental access-before-initialization bugs.

---

## Q4. Explain JavaScript execution context.

### Interview answer

An execution context is the environment in which JavaScript code executes. At a high level, JavaScript starts with a global execution context and creates additional function execution contexts when functions are called.

Conceptually each context contains information about the currently executing code, its variables and lexical environment, and execution state.

```text
Global Execution Context
        |
        v
Function Execution Context
        |
        v
Nested Function Context
```

You should understand the conceptual phases:

```text
Creation / setup
      ↓
Execution
```

Avoid presenting this as a literal two-step algorithm with every engine implementation detail fixed; the model is primarily useful for reasoning about scope, hoisting, and function execution.

---

## Q5. What is the call stack?

### Interview answer

The call stack tracks active JavaScript execution frames. When a function is called, a frame is pushed onto the stack. When it returns, that frame is removed.

```js
function first() {
  second();
}

function second() {
  console.log('Hello');
}

first();
```

Conceptually:

```text
second()
first()
global()
```

Then the frames unwind.

### Why is it important in React Native?

A long-running synchronous calculation on the JS thread can prevent other JavaScript work from running promptly, which can contribute to interaction and rendering problems.

---

## Q6. What is a closure?

### Interview answer

A closure occurs when a function retains access to variables from its lexical scope even after the outer function has returned.

```js
function counter() {
  let count = 0;

  return function increment() {
    count += 1;
    return count;
  };
}

const increment = counter();

console.log(increment()); // 1
console.log(increment()); // 2
```

`increment` closes over `count`.

### React Native relevance

Closures appear in:

- event handlers
- `useEffect`
- `useCallback`
- timers
- promise callbacks
- custom hooks
- state updater functions

Understanding closures is essential for debugging **stale state**.

---

## Q7. What is a stale closure in React?

### Interview answer

A stale closure happens when a callback retains values from an older render and later executes with those captured values instead of the latest state or props.

```js
function Screen() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setTimeout(() => {
      console.log(count);
    }, 3000);
  };

  // ...
}
```

The timeout callback captures `count` from the render where that callback was created.

### How do you handle it?

Use the right strategy for the problem.

When calculating new state from previous state:

```js
setCount(prev => prev + 1);
```

When an imperative callback genuinely needs a mutable reference to the latest value:

```js
const countRef = useRef(count);
countRef.current = count;
```

### Senior point

Do not use refs automatically to fix every stale closure. First determine whether the code should instead have correct dependencies or use a functional state update.

---

## Q8. Explain `this` in JavaScript.

### Interview answer

For normal functions, the value of `this` is primarily determined by how the function is called.

```js
const user = {
  name: 'Surendra',
  greet() {
    console.log(this.name);
  },
};

user.greet(); // this -> user
```

If the method is extracted, the receiver can be lost:

```js
const greet = user.greet;
greet();
```

Arrow functions behave differently: they do not create their own `this`; they use the lexical `this` from the surrounding scope.

### Senior follow-up

Know the common call patterns:

```text
obj.method()       → this is obj
func.call(obj)     → explicit this
func.apply(obj)    → explicit this
func.bind(obj)     → returns bound function
new Func()         → this refers to created instance
arrow function     → lexical this
```

---

## Q9. What is the difference between `call`, `apply`, and `bind`?

### Interview answer

All three can control `this` for normal functions.

```js
function greet(city) {
  console.log(this.name, city);
}

const user = { name: 'Surendra' };

greet.call(user, 'Coimbatore');
greet.apply(user, ['Coimbatore']);

const bound = greet.bind(user);
bound('Coimbatore');
```

- `call` invokes immediately with arguments separately.
- `apply` invokes immediately with arguments as an array-like collection.
- `bind` returns a new function with `this` and optional leading arguments bound.

---

## Q10. What is the prototype chain?

### Interview answer

JavaScript uses prototype-based inheritance. If a property is not found directly on an object, the runtime looks up the object's prototype and continues through the prototype chain until it finds the property or reaches `null`.

```text
object
  ↓
prototype
  ↓
prototype
  ↓
Object.prototype
  ↓
null
```

Methods such as `toString` are typically found through this chain rather than being duplicated on every object instance.

---

## Q11. Are JavaScript classes actually classical inheritance?

### Interview answer

JavaScript provides `class` syntax and inheritance constructs, but the language itself remains prototype-based. Classes provide a more familiar syntax over prototype mechanisms rather than replacing them with a purely classical object model.

```js
class Animal {
  speak() {
    console.log('sound');
  }
}

class Dog extends Animal {
  bark() {
    console.log('bark');
  }
}
```

A senior interviewer may ask what `extends` ultimately establishes: it sets up a prototype relationship between the child and parent constructor/prototype objects.

---

## Q12. Shallow copy vs deep copy.

### Shallow copy

```js
const user = {
  name: 'A',
  address: {
    city: 'Chennai',
  },
};

const copy = { ...user };
```

`copy.address` still references the same nested object.

### Deep copy

For cloneable data, modern JavaScript provides:

```js
const cloned = structuredClone(user);
```

Other strategies or libraries may be appropriate for special object types.

### React relevance

For nested state updates, copy each changed level:

```js
setUser(prev => ({
  ...prev,
  address: {
    ...prev.address,
    city: 'Coimbatore',
  },
}));
```

---

## Q13. What is immutability and why is it important in React?

### Interview answer

Immutability means you avoid changing an existing state value in place and instead produce a new value representing the next state. React and many state libraries rely on reference identity for efficient change detection and memoization.

Bad:

```js
user.name = 'Kumar';
setUser(user);
```

Better:

```js
setUser(prev => ({
  ...prev,
  name: 'Kumar',
}));
```

### Important nuance

Immutability is not the same as deep-freezing every object. The practical goal is predictable state transitions and stable identity semantics.

---

## Q14. Difference between `==` and `===`.

```js
5 == '5';  // true
5 === '5'; // false
```

### Interview answer

`==` performs type coercion as part of comparison. `===` compares without the usual coercive conversion of `==`. In application code, `===` is generally preferred because its behavior is more predictable.

Know a few edge cases, but don't fill an interview with coercion trivia unless asked.

---

## Q15. Explain the JavaScript event loop.

### Interview answer

Synchronous JavaScript executes on the call stack. Asynchronous operations are coordinated by the runtime. When asynchronous work is ready, its continuation is queued, and the event loop decides when queued work can run after the current execution stack is clear.

Conceptually:

```text
                JavaScript
                    |
               Call Stack
                    |
        -------------------------
        |                       |
   Microtask Queue          Task Queue
        |                       |
        -------- Event Loop -----
```

Example:

```js
console.log('A');

setTimeout(() => {
  console.log('B');
}, 0);

Promise.resolve().then(() => {
  console.log('C');
});

console.log('D');
```

Typical order in environments with standard promise microtask scheduling:

```text
A
D
C
B
```

The exact task scheduling model depends on the JavaScript host, but the core interview concept is that Promise reactions run as microtasks and timers are handled as tasks.

---

## Q16. What is the difference between microtasks and macrotasks/tasks?

Examples commonly treated as microtasks:

```text
Promise.then / catch / finally
queueMicrotask
```

Examples of tasks/macrotask-style APIs:

```text
setTimeout
setInterval
message events and other host tasks
```

General mental model:

```text
Current synchronous work
        ↓
Microtasks
        ↓
Next scheduled task
```

### Interview trap

`setTimeout(fn, 0)` does not mean “run immediately.” It means the callback becomes eligible to run after the current work and according to the host's scheduling rules.

---

## Q17. Explain Promise states.

A Promise is initially:

```text
pending
```

It eventually becomes one of:

```text
fulfilled
rejected
```

Once settled, it cannot transition to the other final state.

```text
pending
  |
  +---- fulfilled
  |
  +---- rejected
```

---

## Q18. Difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`.

### `Promise.all`

Waits for all inputs. Rejects when one rejects.

```js
await Promise.all([
  fetchUser(),
  fetchProducts(),
  fetchOrders(),
]);
```

Use when all operations are required for success.

### `Promise.allSettled`

Waits for every Promise and provides each result status. Useful when partial success matters.

### `Promise.race`

Settles when the first input settles, whether fulfilled or rejected.

### `Promise.any`

Fulfills as soon as the first input fulfills. It rejects only if all inputs reject.

### Interview follow-up

Ask yourself whether the API design really requires serial execution or whether independent requests can be parallelized safely.

---

## Q19. What does `async/await` actually do?

### Interview answer

`async/await` is syntax built around Promise-based asynchronous programming. An `async` function always returns a Promise. `await` suspends the continuation of that async function until the awaited Promise settles; it does not block the whole JavaScript runtime in the same way a synchronous blocking operation would.

```js
async function getData() {
  const response = await fetch(url);
  return response.json();
}
```

### Senior follow-up

Know when to parallelize:

```js
const [user, products] = await Promise.all([
  getUser(),
  getProducts(),
]);
```

instead of unnecessarily doing:

```js
const user = await getUser();
const products = await getProducts();
```

when the operations are independent.

---

## Q20. Is `await` blocking?

### Interview answer

It suspends the current async function's continuation. It does not block unrelated JavaScript execution in the general sense, and it is not equivalent to a thread-blocking synchronization primitive.

A stronger answer than “`await` blocks JavaScript” is:

> `await` pauses that function's async continuation while control returns to the runtime until the Promise settles.

---

## Q21. What is debouncing?

Debouncing delays execution until a period of inactivity has passed.

Search example:

```text
R
Re
Rea
React
   ↓
(wait)
   ↓
API call
```

Typical use cases:

- search boxes
- validation
- window resizing
- autosave triggers

---

## Q22. What is throttling?

Throttling limits how often a function can execute within a time window.

Typical use cases:

- scroll events
- gesture-related work
- analytics sampling
- frequent sensor/location updates

Difference:

```text
Debounce → wait until activity settles
Throttle → execute at controlled intervals
```

---

## Q23. How does garbage collection work?

### Interview answer

JavaScript runtimes automatically reclaim memory that is no longer reachable. Modern engines use garbage-collection strategies to identify objects that are no longer needed and reclaim their memory.

For React Native, resource cleanup still matters even if JavaScript memory is garbage-collected. Native handles, listeners, timers, subscriptions, sockets, and other resources may require explicit cleanup.

Example:

```js
useEffect(() => {
  const subscription = subscribe();

  return () => {
    subscription.remove();
  };
}, []);
```

---

## Q24. What causes memory leaks in React Native?

Common causes include:

- timers that are never cleared
- event listeners/subscriptions that are never removed
- WebSockets that remain open
- callbacks that retain unexpectedly large object graphs
- large image/data caches
- native resources that aren't released correctly
- long-lived stores holding unnecessary data

A practical debugging strategy is to reproduce the screen transition repeatedly and inspect whether memory or active resources continuously grow.

---

# Part 2 — React Fundamentals

## Q25. What is React?

### Interview answer

React is a library for building user interfaces with a declarative component model. You describe the UI for a given state, and React coordinates the process of producing the next rendered tree when state or props change.

For React Native:

```text
React components
      ↓
React Native renderer
      ↓
Native platform UI
```

---

## Q26. What is the Virtual DOM?

### Interview answer

The Virtual DOM is a conceptual representation of the UI tree used by React's rendering model. When a component renders again, React produces a new element tree and determines what must change relative to the previous tree before committing updates.

Avoid saying:

> React rebuilds and compares the entire real DOM every time.

That oversimplifies React's reconciliation process.

In React Native there is no browser DOM equivalent; the same React rendering model is paired with native renderers.

---

## Q27. What is reconciliation?

### Interview answer

Reconciliation is the process by which React determines how a new rendered element tree relates to the previous tree and which component identities and updates should be preserved, replaced, or changed.

Key concepts include:

- element/component type
- stable keys in lists
- component identity
- tree structure
- props and state changes

---

## Q28. What is the difference between render phase and commit phase?

### Render phase

React computes what the next UI should look like.

### Commit phase

React applies the necessary updates to the host environment.

Conceptually:

```text
State update
    ↓
Render
    ↓
Reconciliation
    ↓
Commit
    ↓
Host UI update
```

### Senior point

Render work should be treated as pure. Do not trigger network requests, subscriptions, or other imperative side effects from the component render function.

---

## Q29. Why should side effects not happen during render?

Because a render is supposed to be a pure calculation of UI. React may render more than once, restart work, or abandon a render before it commits.

Bad:

```js
function UserScreen() {
  fetchUser();
  return <View />;
}
```

Better:

```js
useEffect(() => {
  fetchUser();
}, [userId]);
```

Depending on the operation, an event handler or data-fetching abstraction can be even more appropriate than an effect.

---

## Q30. What is a React component?

### Interview answer

A component is a reusable UI unit that accepts inputs such as props and returns a description of what should be rendered.

```jsx
function UserCard({ user }) {
  return (
    <View>
      <Text>{user.name}</Text>
    </View>
  );
}
```

A component can own local state and coordinate hooks, but it should avoid becoming an unstructured container for every business rule in the application.

---

## Q31. Props vs state.

| Props | State |
|---|---|
| Inputs from a parent/owner | Data owned by a component or state system |
| Read-only from the receiving component's perspective | Changes over time |
| Used to configure/render a child | Represents dynamic application/UI state |

Example prop:

```jsx
<UserCard user={user} />
```

Example state:

```js
const [loading, setLoading] = useState(false);
```

---

## Q32. What happens when `setState` is called?

### Interview answer

A state update schedules React to process an update. React may batch updates, compute the next state, render affected components, and commit the resulting changes.

For state derived from previous state, use a functional updater:

```js
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

This lets each updater receive the current queued state value rather than relying on a captured variable from the surrounding render.

---

## Q33. Why use functional state updates?

When the next value depends on the previous value:

```js
setCount(prev => prev + 1);
```

It is especially useful for:

- multiple updates
- batched updates
- timers
- asynchronous callbacks
- event handlers that may run after the current render

---

## Q34. Explain `useEffect`.

### Interview answer

`useEffect` is used to synchronize a component with external systems or imperative resources that are not part of rendering itself. Examples include subscriptions, timers, network synchronization, event listeners, and other APIs.

```js
useEffect(() => {
  const subscription = subscribe();

  return () => {
    subscription.remove();
  };
}, []);
```

### Senior point

Not every piece of logic belongs in an effect. Derived values should usually be calculated during render, while user-triggered actions often belong in event handlers.

---

## Q35. What does the dependency array mean?

```js
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

The effect reads `userId`, so changes to that reactive value are relevant to the effect. React compares dependency values between renders and reruns the effect when they have changed.

A dependency array is **not** an arbitrary “when I want this effect to run” list. It should represent the reactive values the effect depends on, with deliberate exceptions when using established patterns.

---

## Q36. What is wrong with this?

```js
useEffect(() => {
  fetchUser(userId);
}, []);
```

If `userId` can change, the effect may keep using the value captured from the initial render.

A good interview answer:

> I would check whether `userId` is actually stable. If it is reactive, it should normally be represented in the dependencies or the logic should be refactored so that the effect's dependencies accurately describe what it synchronizes with.

---

## Q37. What does `useEffect` cleanup do?

```js
useEffect(() => {
  const listener = addListener();

  return () => {
    listener.remove();
  };
}, []);
```

Cleanup runs before a subsequent execution of the same effect when dependencies change and when the effect is removed/unmounted according to React's lifecycle semantics.

Use cleanup for:

- event listeners
- timers
- subscriptions
- WebSockets
- native observers
- request cancellation where appropriate

---

## Q38. Explain `useMemo`.

### Interview answer

`useMemo` caches the result of a calculation between renders until its dependencies change.

```js
const filteredProducts = useMemo(() => {
  return products.filter(product =>
    product.name.includes(search)
  );
}, [products, search]);
```

Use it when:

- a calculation is meaningfully expensive, or
- stable reference identity is valuable for downstream optimization.

Do not use it as a blanket rule for every variable. Memoization itself has cost and complexity.

---

## Q39. Explain `useCallback`.

### Interview answer

`useCallback` returns the same function reference between renders until one of its dependencies changes.

```js
const handlePress = useCallback(() => {
  navigation.navigate('Details');
}, [navigation]);
```

It is most useful when reference stability matters, for example:

- passing a callback to a memoized child
- using a callback in dependency tracking
- avoiding needless downstream prop changes in an expensive subtree

It does not make the underlying function intrinsically faster.

---

## Q40. `useMemo` vs `useCallback`.

```text
useMemo
→ memoizes a calculated value

useCallback
→ memoizes a function reference
```

Conceptually:

```js
useCallback(fn, deps)
```

is closely related to:

```js
useMemo(() => fn, deps)
```

but the semantic intent is clearer with `useCallback` when the thing being memoized is a function.

---

## Q41. What is `React.memo`?

### Interview answer

`React.memo` creates a memoized component that can skip a re-render when its props compare as unchanged.

```js
const UserCard = React.memo(function UserCard({ user }) {
  return <Text>{user.name}</Text>;
});
```

However, this can be undermined by unstable props:

```jsx
<UserCard
  user={user}
  onPress={() => handlePress(user)}
/>
```

The callback is a new function on each parent render.

---

## Q42. Does `React.memo` guarantee no re-render?

No.

### Interview answer

`React.memo` is a performance optimization, not a semantic guarantee. A component can still render because its own state changes, consumed context changes, props change, or React otherwise needs to render it.

Also, preventing every render is not inherently the goal. The goal is to prevent unnecessary expensive work.

---

## Q43. What is the Context API?

### Interview answer

Context allows a value to be provided to a subtree without explicitly passing it through every intermediate component as a prop.

Common uses:

- theme
- localization
- authentication dependencies
- application configuration
- relatively stable cross-cutting data

```jsx
<AuthProvider>
  <App />
</AuthProvider>
```

---

## Q44. Should Context replace Redux?

Not automatically.

A strong distinction is:

```text
Context
→ dependency propagation / shared values

Redux
→ centralized application state and predictable state transitions
```

Context is excellent for relatively stable cross-cutting dependencies. Redux or another store can be more appropriate when state is large, frequently updated, shared across many features, or benefits from centralized transitions, middleware, persistence, or debugging tooling.

---

## Q45. What causes Context performance problems?

Consider:

```jsx
<AuthContext.Provider value={{ user, login }}>
  {children}
</AuthContext.Provider>
```

If the provider renders and creates a fresh object, consumers may observe a new context value.

Possible mitigation:

```js
const value = useMemo(
  () => ({ user, login }),
  [user, login]
);
```

However, the bigger architectural consideration is provider scope. Avoid putting high-frequency changing values into very broad contexts when more localized state would be better.

---

## Q46. What is `useRef`?

### Interview answer

`useRef` stores a mutable value that persists across renders without causing a re-render when the `.current` value changes.

Example:

```js
const inputRef = useRef(null);

inputRef.current?.focus();
```

It can also hold:

- timer IDs
- previous values
- imperative handles
- mutable instance-like state
- latest values for carefully designed callbacks

---

## Q47. `useState` vs `useRef`.

| `useState` | `useRef` |
|---|---|
| Updating state schedules rendering | Changing `.current` does not schedule rendering |
| Represents UI/application state | Represents a persistent mutable cell |
| State transitions are part of React's update model | Mutations are imperative and not themselves rendered |

Use a ref when rendering should **not** be triggered by the value changing.

---

## Q48. What are the Rules of Hooks?

The core rules are:

1. Call hooks only at the top level of a React function component or custom hook.
2. Call hooks only from React components or custom hooks.

Bad:

```js
if (isLoggedIn) {
  useEffect(() => {}, []);
}
```

Hook order must remain consistent across renders so React can correctly associate hook state with hook calls.

---

## Q49. What is a custom hook?

### Interview answer

A custom hook is a reusable function beginning with `use` that can compose React hooks and encapsulate reusable stateful behavior.

```js
function useAuth() {
  const [user, setUser] = useState(null);

  const login = async credentials => {
    // login logic
  };

  return { user, login };
}
```

The goal is to separate reusable behavior from UI composition.

---

## Q50. Why are keys important in React lists?

Keys give elements stable identity so React can reason about inserted, removed, or reordered items.

```jsx
products.map(product => (
  <ProductCard
    key={product.id}
    product={product}
  />
));
```

Use stable IDs whenever possible.

---

## Q51. Why can using array index as a key cause bugs?

Imagine:

```text
A
B
C
```

Then insert `X` at the top:

```text
X
A
B
C
```

Indexes shift. React may associate an existing component instance with different data than before.

This can lead to:

- wrong local input state
- mismatched component state
- unexpected animations
- confusing list updates

Index keys are not inherently forbidden; they are risky when items can be reordered, inserted, removed, or otherwise change identity.

---

## Q52. What is Strict Mode?

### Interview answer

React Strict Mode enables additional development-time checks intended to surface unsafe patterns. In development, some behavior can be repeated or simulated to expose missing cleanup and side-effect bugs. This should not be interpreted as a promise that production executes the exact same development checks.

When an API appears to run twice in development, inspect:

- Strict Mode behavior
- effect dependencies
- mount/unmount behavior
- cleanup
- duplicate invocation from app code

---

## Q53. What is lazy loading?

```js
const Profile = lazy(() => import('./Profile'));
```

Combined with:

```jsx
<Suspense fallback={<Loader />}>
  <Profile />
</Suspense>
```

The module is loaded when React needs it according to the surrounding code-splitting/runtime model.

For mobile, evaluate whether code splitting materially improves startup and whether the resulting loading behavior is appropriate for the product.

---

## Q54. What is an Error Boundary?

### Interview answer

An Error Boundary is a component that catches errors during rendering and lifecycle behavior in descendant components and shows fallback UI instead of allowing the affected subtree to fail unhandled.

An error boundary is not a general-purpose asynchronous exception handler. Network errors, promise rejections, timers, and event callback failures often require their own handling patterns.

---

# Part 3 — React Native Fundamentals

## Q55. What is React Native?

### Interview answer

React Native is a framework for building native mobile applications using React and JavaScript/TypeScript. React supplies the declarative component model, while React Native provides host components, a renderer, and APIs for interacting with Android and iOS capabilities.

Example:

```jsx
<View>
  <Text>Hello</Text>
</View>
```

These are React Native components, not HTML DOM elements.

---

## Q56. React vs React Native.

| React (web) | React Native |
|---|---|
| Browser UI | Native mobile UI |
| DOM host environment | Native host environment |
| `div` | `View` |
| `button` | `Pressable` / platform controls |
| Browser CSS | React Native style system + Yoga layout |
| Browser APIs | Native/mobile APIs |

The programming model is React, but the host renderer and platform capabilities are different.

---

## Q57. Explain React Native architecture.

You should understand both the historical architecture and the modern New Architecture.

### Legacy architecture

A simplified mental model is:

```text
JavaScript
    |
    | serialized async Bridge communication
    ↓
Native
```

### New Architecture

A simplified mental model:

```text
React
  ↓
Fabric renderer
  ↓
Native rendering

JavaScript
  ↓
JSI
  ↓
TurboModules / native capabilities
  ↓
Native
```

Important New Architecture concepts:

- Fabric
- TurboModules
- JSI
- concurrent React capabilities / improved scheduling integration
- code generation and typed native interfaces in the wider architecture ecosystem

Do not describe the New Architecture as “just replacing the bridge with JSI.” It is a broader redesign of the rendering and native module systems.

---

## Q58. What is the React Native Bridge?

### Interview answer

In the legacy React Native architecture, the Bridge was a communication mechanism between JavaScript and native code. Communication commonly involved batching/serialization and asynchronous transfer across the boundary.

This could become expensive for very high-frequency communication such as:

- gesture updates
- animation values
- large event streams
- frequent native callbacks

The New Architecture moves away from this legacy communication model and uses JSI-based mechanisms plus modern rendering/native module systems.

---

## Q59. What is JSI?

### Interview answer

JSI, the JavaScript Interface, is a C++ interface that allows JavaScript engines and native/C++ code to interact more directly than the old serialized Bridge model.

Conceptually:

```text
Old
JS → Bridge → Native

Modern
JS ↔ JSI ↔ C++ / Native infrastructure
```

JSI enables libraries and React Native internals to implement lower-overhead interactions, including synchronous/native-adjacent patterns where appropriate.

---

## Q60. What are TurboModules?

### Interview answer

TurboModules are part of React Native's New Architecture for integrating native modules with JavaScript. They support a more modern module-loading and interoperability model, including JSI-based communication patterns and lazy loading.

Examples of native capabilities that may be exposed as modules include:

- camera
- biometrics
- Bluetooth
- device sensors
- secure storage
- payments

---

## Q61. What is Fabric?

### Interview answer

Fabric is React Native's modern rendering system. It is part of the New Architecture and is designed to improve how React's rendering model integrates with native views and modern React capabilities.

Conceptually:

```text
React tree
   ↓
Fabric renderer
   ↓
Native host views
```

When discussing Fabric, focus on its role as the renderer rather than describing it simply as a performance toggle.

---

## Q62. What is Hermes?

### Interview answer

Hermes is a JavaScript engine optimized for React Native workloads, with a focus on mobile constraints such as startup performance and memory usage.

A useful mental model is:

```text
React Native application
      ↓
JS bundle
      ↓
JavaScript engine (commonly Hermes)
      ↓
React Native runtime / renderer
```

Know how Hermes can affect debugging, stack traces, startup characteristics, and runtime behavior in production.

---

## Q63. What are the JavaScript thread and UI/main thread?

A simplified model is:

```text
JS thread / runtime
  ├─ React/business logic
  ├─ state processing
  ├─ API callbacks
  └─ JavaScript computations

UI/Main thread
  ├─ native UI work
  ├─ layout and drawing
  └─ user interaction handling
```

A long-running synchronous JavaScript computation can delay JS-side work and make the app feel unresponsive.

Do not present this model as if React Native has only exactly two threads. Modern RN apps can involve multiple runtime/native threads. The model is useful for identifying whether work is JS-bound or native/UI-bound.

---

## Q64. How would you debug UI lag in React Native?

### Strong 5+ years answer

First determine the bottleneck instead of applying random optimizations. I would establish whether the issue is:

- JavaScript execution
- UI/native work
- excessive renders
- expensive component work
- list virtualization
- image decoding
- animation/gesture processing
- networking/data transformation
- memory pressure

Then profile the interaction and compare before/after changes.

A good diagnostic flow is:

```text
Reproduce
  ↓
Profile
  ↓
Identify bottleneck
  ↓
Target optimization
  ↓
Measure again
```

---

## Q65. Why is `FlatList` preferred over `ScrollView` for large lists?

### Interview answer

`ScrollView` generally renders all of its children, whereas `FlatList` uses virtualization to render a window of items around the visible area and manage item recycling/rendering more efficiently.

For large datasets, use:

```jsx
<FlatList
  data={products}
  renderItem={renderItem}
  keyExtractor={item => String(item.id)}
/>
```

rather than rendering a very large array directly inside a `ScrollView`.

---

## Q66. Important `FlatList` optimization techniques.

Know these and, more importantly, know when to use them:

- stable `keyExtractor`
- lightweight row components
- `React.memo` for expensive rows when appropriate
- stable `renderItem` references when it matters
- `getItemLayout` for fixed-size items
- pagination/infinite loading
- sensible `initialNumToRender`
- `maxToRenderPerBatch`
- `windowSize`
- careful use of `removeClippedSubviews`
- image sizing/caching
- avoid large derived calculations in every row
- avoid passing constantly changing objects/functions unnecessarily

Do not present these as a magic checklist. Profiling is more important than changing ten props at once.

---

## Q67. What is `getItemLayout`?

For fixed or predictable row dimensions, `getItemLayout` lets the list calculate item positions without dynamically measuring every item.

```js
getItemLayout={(data, index) => ({
  length: ITEM_HEIGHT,
  offset: ITEM_HEIGHT * index,
  index,
})}
```

This can improve operations such as jumping to an item by index and reduce measurement work.

---

## Q68. `FlatList` vs `SectionList`.

### `FlatList`

For a single flat sequence:

```text
Products
Users
Messages
```

### `SectionList`

For grouped/sectioned data:

```text
A
  Apple
  Amazon

B
  BMW
  Boeing
```

---

## Q69. What is `VirtualizedList`?

### Interview answer

`VirtualizedList` is the lower-level virtualization primitive used by `FlatList` and `SectionList`. It provides more direct control for advanced cases where data access or list behavior differs from the standard array-based components.

---

## Q70. How can you prevent unnecessary React Native renders?

Start with architecture before micro-optimizations:

1. Keep state as local as practical.
2. Create useful component boundaries.
3. Avoid recreating expensive data structures unnecessarily.
4. Use `React.memo` when it meaningfully reduces expensive child work.
5. Use `useCallback` when function identity matters.
6. Use `useMemo` for expensive calculations or important stable references.
7. Avoid expensive work during render.
8. Virtualize large collections.
9. Avoid broad context updates for high-frequency state.
10. Optimize selectors and derived data when using state libraries.

---

## Q71. LayoutAnimation / Animated vs Reanimated.

The exact APIs differ by version, but the conceptual distinction is important.

React Native provides animation primitives, while Reanimated is designed for advanced, highly interactive animations and gestures where running work closer to the UI/native execution side can reduce JS-thread pressure.

Typical Reanimated use cases:

- gesture-driven interactions
- shared values
- complex transitions
- high-frequency visual updates
- UI-thread-oriented animation work

The senior-level point is:

> High-frequency visual work should not unnecessarily depend on a busy JavaScript thread.

---

## Q72. Why avoid expensive calculations inside render?

Consider:

```jsx
function ProductList({ products }) {
  const result = hugeCalculation(products);
  return <FlatList data={result} />;
}
```

If the component renders frequently, the calculation repeats.

A potential optimization is:

```js
const result = useMemo(
  () => hugeCalculation(products),
  [products]
);
```

Only use this if the calculation is meaningful enough to justify memoization.

---

## Q73. What is `Pressable` and why might you prefer it over older touchables?

### Interview answer

`Pressable` is a React Native core component designed around press interaction states and gives more control over pressed behavior.

It is useful for reusable interaction components because you can derive styling from pressed state.

```jsx
<Pressable
  onPress={handlePress}
  style={({ pressed }) => [
    styles.button,
    pressed && styles.pressed,
  ]}
>
  <Text>Submit</Text>
</Pressable>
```

The main interview point is to understand touch interaction, accessibility, hit areas, and platform behavior rather than memorizing a component name.

---

# Part 4 — Navigation

## Q74. React Navigation vs Expo Router.

### Interview answer

React Navigation provides navigation primitives and configuration patterns for React Native. Expo Router adds file-based routing and deep integration with the Expo toolchain.

Expo Router conventionally maps filesystem structure to routes:

```text
app/
├── index.tsx
├── login.tsx
├── profile.tsx
└── products/
    └── [id].tsx
```

The right choice depends on the project's routing conventions, web support needs, team preferences, and existing architecture.

---

## Q75. How would you implement authentication navigation?

Typical model:

```text
                    Root
                      |
             Authenticated?
              /           \
            No             Yes
            |                |
         AuthStack        AppStack
            |                |
          Login          Home/Profile
```

The key is that access control should be driven by actual authentication/session state, not simply by visually hiding screens.

With Expo Router, this commonly maps to public/authenticated route groups and an auth-aware root layout.

---

## Q76. How do deep links work?

Example:

```text
myapp://products/123
```

or web-based app links/universal links:

```text
https://example.com/products/123
```

The application maps:

```text
URL
 ↓
route
 ↓
screen
 ↓
parameters
```

Production concerns include:

- cold start vs warm start
- authentication state
- expired or invalid links
- nested navigation
- Android App Links
- iOS Universal Links
- navigation state restoration

---

# Part 5 — Expo and EAS

## Q77. What is Expo?

### Interview answer

Expo is a framework and ecosystem around React Native that provides standardized tooling for app development, native capabilities, builds, updates, and deployment workflows. Expo projects can also use custom native modules and native configuration through development builds, prebuild/config plugins, and native project code when necessary.

A good modern interview answer avoids describing Expo as a “beginner-only wrapper.”

---

## Q78. What is Expo Go?

### Interview answer

Expo Go is a prebuilt development client containing a predefined set of Expo capabilities. It is convenient for learning and projects that fit within its supported native surface, but it is not a substitute for a project-specific development build when custom native modules or native configuration are required.

---

## Q79. Expo Go vs Development Build.

| Expo Go | Development Build |
|---|---|
| Prebuilt client | Project-specific native client |
| Limited to included native capabilities | Can include custom native modules |
| Very quick to start | More closely matches your native app |
| Excellent for learning/prototyping | Better for serious native customization |

A development build is an installable debug-oriented application binary tailored to your project.

---

## Q80. What is `expo prebuild`?

### Interview answer

`expo prebuild` generates native Android and iOS projects from the Expo configuration and the project's native module requirements.

Conceptually:

```text
app.json / app.config.js
       +
Expo/native packages
       ↓
expo prebuild
       ↓
android/ + ios/
```

It is particularly relevant when native project configuration is needed while keeping configuration-driven reproducibility.

---

## Q81. What are Expo config plugins?

### Interview answer

Config plugins allow packages or app configuration to modify native project settings during prebuild instead of requiring every native modification to be done manually in generated files.

Conceptually:

```text
Expo config
   ↓
Config plugin
   ↓
Native project modifications
   ├─ AndroidManifest
   ├─ Gradle settings
   ├─ Info.plist
   └─ Xcode project configuration
```

They are important for reproducible native configuration.

---

## Q82. What is EAS Build?

### Interview answer

EAS Build is Expo's hosted build service for creating Android and iOS application binaries. It supports build profiles, signing credentials, internal distribution, and automated workflows.

Example:

```bash
eas build --platform all
```

In an interview, mention that build profiles let teams separate development, preview/QA, and production behavior.

---

## Q83. What is EAS Update?

### Interview answer

EAS Update allows compatible JavaScript and asset changes to be delivered to existing application binaries without producing a new native binary for every JS/assets-only change.

Conceptually:

```text
Existing binary
    |
    +── Native runtime
    +── JS/assets
          ↑
       EAS Update
```

The update still must be compatible with the binary's native runtime. `runtimeVersion` is part of the compatibility model.

---

## Q84. Can EAS Update replace Play Store/App Store releases?

No.

### Interview answer

OTA updates are not a replacement for native releases. If you change native code, native dependencies, native configuration, or anything that changes the runtime contract, a new application binary is generally required.

For compatible JS/assets changes, an OTA update can avoid a full store release.

---

## Q85. What are EAS build profiles?

Example:

```json
{
  "build": {
    "development": {},
    "preview": {},
    "production": {}
  }
}
```

Typical roles:

```text
development
→ developer iteration / development client

preview
→ QA / internal distribution

production
→ store/release binaries
```

Profiles can also control environment variables, distribution mode, channel, simulator/device settings, and other build behavior.

---

## Q86. How would you manage app icons, names, bundle IDs, and other app branding in Expo?

For a configurable multi-brand application, define branding as build-time configuration rather than runtime secret logic.

Conceptually:

```text
Brand configuration
      ↓
Expo app config
      ↓
EAS build profile / environment
      ↓
app name, icons, package/bundle ID
      ↓
separate native binary
```

The runtime can also receive API-driven content, but native app identity elements such as the launcher icon and bundle identifier are generally build-time concerns, not values that can be safely changed inside an already-installed binary by downloading JSON.

---

# Part 6 — State Management

## Q87. When would you use Redux?

### Interview answer

I use Redux when shared application state becomes complex enough that centralized predictable updates, middleware, persistence, debugging, cross-feature coordination, or explicit state transitions provide real value.

Examples:

- authentication/session state
- cart
- favorites
- user preferences
- feature flags
- complex multi-screen workflows

I would not introduce Redux simply because an app has more than one component.

---

## Q88. Redux Toolkit vs legacy Redux.

Redux Toolkit is the recommended approach for modern Redux applications. It reduces boilerplate and standardizes patterns with utilities such as:

- `configureStore`
- `createSlice`
- `createAsyncThunk`
- RTK Query

Example:

```js
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload);
    },
  },
});
```

The mutation-style code is translated into immutable updates by Immer.

---

## Q89. Is Redux Toolkit actually mutating state?

### Interview answer

The reducer code can look mutative because Redux Toolkit uses Immer. Immer tracks the changes and produces an immutable next state.

So:

```js
state.items.push(item);
```

is mutation-like reducer syntax, but the resulting Redux state update remains immutable from the store's perspective.

---

## Q90. Context vs Redux.

A useful interview distinction:

```text
Context
→ dependency propagation

Redux
→ centralized application state
```

Context is good for relatively stable cross-cutting dependencies. Redux is useful when complex shared state needs a predictable update model, middleware, persistence, or debugging.

---

## Q91. What is Redux middleware?

### Interview answer

Middleware runs between dispatch and reducer processing.

```text
dispatch(action)
       ↓
middleware
       ↓
reducer
       ↓
store update
       ↓
UI
```

Use cases include:

- async workflows
- analytics
- logging
- error reporting
- side effects
- conditional dispatching

---

## Q92. What is RTK Query?

### Interview answer

RTK Query is Redux Toolkit's data-fetching and caching solution. It addresses server-state concerns such as request lifecycle, caching, deduplication, invalidation, refetching, and derived loading/error status.

A useful conceptual distinction is:

```text
Client/application state
→ Redux slices

Server/API state
→ RTK Query or another server-state cache
```

---

## Q93. Should all API data be stored in Redux?

No.

### Interview answer

The right home for data depends on its lifecycle. Server state often belongs in a dedicated caching/data-fetching layer such as RTK Query. Local UI state can stay in a component. Cross-feature client state may belong in Redux. Storing every response in a global store can create unnecessary complexity and duplication.

---

## Q94. What are selectors and why are they important?

Selectors read data from state and can centralize derived-state logic.

Example:

```js
const selectCartTotal = state =>
  state.cart.items.reduce(
    (total, item) => total + item.price * item.quantity,
    0
  );
```

For complex derived data, memoized selectors can prevent unnecessary recalculation and help control rerenders.

---

# Part 7 — Networking and API Architecture

## Q95. How do you structure API calls in a large React Native application?

Avoid scattering raw HTTP calls throughout every screen.

Prefer layers such as:

```text
Screen
  ↓
Hook / state abstraction
  ↓
Service / repository
  ↓
API client
  ↓
Backend
```

Typical folders:

```text
src/
├── api/
├── services/
├── hooks/
├── store/
└── features/
```

The exact naming is less important than separation of concerns.

---

## Q96. How would you implement access-token refresh?

Typical flow:

```text
API request
    ↓
401 Unauthorized
    ↓
Access token expired?
    ↓
Refresh token
    ↓
New access token
    ↓
Retry original request
```

Production concerns:

- refresh request must not recursively trigger itself
- multiple failing requests must not start multiple refresh flows unnecessarily
- queued requests should be resolved/rejected correctly
- refresh-token expiry must lead to logout/session reset
- retries need a hard upper bound
- race conditions must be handled

A useful pattern is a shared “refresh in progress” Promise so concurrent requests can await the same refresh operation.

```text
Request A ─┐
Request B ─┼──→ one refresh operation
Request C ─┘
              ↓
           new token
              ↓
       retry queued requests
```

---

## Q97. How do you avoid refresh-token race conditions?

Maintain one refresh operation at a time.

Conceptually:

```js
let refreshPromise = null;

async function getFreshToken() {
  if (!refreshPromise) {
    refreshPromise = refreshAccessToken()
      .finally(() => {
        refreshPromise = null;
      });
  }

  return refreshPromise;
}
```

Every 401 handler can await the same Promise. Production implementations need additional handling for failure, logout, request cancellation, and request retry limits.

---

## Q98. How do you cancel API requests?

Where the HTTP API supports it, use `AbortController`.

```js
const controller = new AbortController();

fetch(url, {
  signal: controller.signal,
});

controller.abort();
```

Useful for:

- search-as-you-type
- abandoning obsolete requests
- screen-specific requests
- avoiding wasted work after navigation

---

## Q99. How do you handle loading, success, empty, and error states?

Do not model everything as one `loading` boolean.

A more explicit state model is:

```text
idle
loading
success + data
success + empty
error + previous data (if applicable)
```

With server-state libraries, these states may be represented separately as fetching/loading/error flags plus cached data.

The UI should distinguish:

```text
first load
background refetch
empty result
network error
permission error
server error
```

---

## Q100. What if the API responds slowly?

Analyze the entire path:

```text
DNS/network
  ↓
server processing
  ↓
response payload
  ↓
transport
  ↓
JSON parsing/data normalization
  ↓
React state update
  ↓
rendering
```

Possible improvements:

- pagination
- smaller payloads
- response compression
- parallel independent requests
- caching
- stale-while-revalidate patterns
- prefetching
- deferred non-critical calls

Do not blame the mobile UI automatically.

---

## Q101. How would you structure an Axios API client?

A typical abstraction:

```js
const api = axios.create({
  baseURL: BASE_URL,
  timeout: 15000,
});

api.interceptors.request.use(config => {
  // attach access token / request metadata
  return config;
});

api.interceptors.response.use(
  response => response,
  async error => {
    // classify errors / refresh / retry as appropriate
    throw error;
  }
);
```

A production-quality client should also consider:

- retry policy
- idempotency
- cancellation
- token refresh recursion
- request correlation IDs
- structured errors
- logging redaction
- environment configuration

---

# Part 8 — Storage and Offline Data

## Q102. AsyncStorage vs SecureStore.

### AsyncStorage

Good for ordinary persistent application data such as:

- preferences
- flags
- simple cache values
- non-sensitive local state

### SecureStore / platform-secure storage

Better for sensitive credentials and secrets that require platform-backed secure storage.

Do not describe generic persistent key-value storage as a secure vault simply because it is inside the app sandbox.

---

## Q103. Where should access tokens be stored?

### Interview answer

Sensitive session credentials should use an appropriate platform-backed secure storage strategy when feasible. The exact approach depends on the token model, threat model, expiry policy, backend capabilities, and whether refresh tokens are used.

Also say:

> The security of a mobile application cannot depend on a secret embedded in the application bundle. Anything shipped to the client should be considered potentially discoverable.

---

## Q104. How would you implement offline support?

Separate:

```text
UI state
Server state
Local persisted state
Sync queue
```

Possible architecture:

```text
UI
 ↓
Local data/cache
 ↓
Sync engine
 ↓
API
```

Important design decisions:

- source of truth
- conflict resolution
- queued writes
- retries
- idempotency
- timestamps/versioning
- cache invalidation
- partial connectivity

For complex offline-first apps, a local database can be more appropriate than a simple key-value store.

---

# Part 9 — Performance

## Q105. A `FlatList` screen is lagging. What do you check?

### Strong answer

I would profile rather than blindly modify props.

Check:

1. Row complexity.
2. Number of rows re-rendering.
3. Unstable callbacks/objects.
4. Large images.
5. Expensive selectors/derived calculations.
6. JS thread workload.
7. UI/native work.
8. Nested lists.
9. Excessively frequent state updates.
10. Animation/gesture workloads.

Then optimize one bottleneck at a time.

---

## Q106. What is a major mistake in React Native performance work?

Premature optimization.

Bad approach:

```text
useMemo everywhere
useCallback everywhere
React.memo everywhere
random FlatList tuning
```

Better:

```text
Measure
 ↓
Identify bottleneck
 ↓
Optimize
 ↓
Measure again
```

---

## Q107. How do images affect performance?

Large images can consume:

- network bandwidth
- memory
- decode time
- GPU resources
- cache space

If a screen displays a 100×100 image but downloads a huge original image, the app can waste resources before the image is displayed.

Use:

- appropriately sized images
- thumbnails
- responsive CDN transformations
- caching
- suitable formats
- lazy loading where appropriate

---

## Q108. How do you optimize application startup?

Think about the complete critical path:

```text
Native initialization
       ↓
JS engine/runtime
       ↓
Bundle loading
       ↓
Root initialization
       ↓
Critical data
       ↓
First meaningful UI
```

Possible improvements:

- reduce startup JS work
- defer non-critical initialization
- avoid unnecessary synchronous work
- reduce dependency/bundle costs
- optimize assets
- cache or precompute critical configuration
- avoid serializing independent initialization requests
- delay secondary analytics/feature initialization

Measure startup rather than guessing.

---

## Q109. How would you improve a slow application launch caused by many API requests?

First classify which requests are truly blocking.

Example:

```text
Critical
→ auth/session restoration
→ minimum home content

Non-critical
→ analytics
→ recommendations
→ secondary widgets
```

Then:

```text
critical calls
   ↓
first meaningful UI
   ↓
background requests
```

Parallelize independent requests and use caching where appropriate.

---

## Q110. How do you diagnose memory pressure?

Look for:

- large image allocations
- retained screen data
- unbounded arrays/caches
- listeners/subscriptions
- native view retention
- repeated screen mount cycles
- large global store objects

Use profiling tools available for the platform and compare memory after repeated navigation cycles.

A useful test is:

```text
Open screen
→ leave screen
→ reopen repeatedly
→ compare memory baseline
```

If the baseline continuously increases without returning near the expected range, investigate retained resources.

---

# Part 10 — Android, iOS, and Native Integration

## Q111. Why do React Native projects contain Android and iOS folders?

Because the delivered application is a native Android/iOS application. JavaScript is one part of the application runtime.

Android includes things such as:

```text
Gradle
Kotlin/Java
AndroidManifest
Resources
```

iOS includes:

```text
Xcode project/workspace
Swift/Objective-C
Info.plist
CocoaPods/native packages
```

---

## Q112. What is Gradle?

### Interview answer

Gradle is the build system used by Android projects. In React Native it handles compilation, dependency resolution, plugins, build variants, signing integration, generated code, and the overall Android build lifecycle.

You should recognize:

```text
Debug
Release
Product Flavors
Build Types
```

and understand that React Native package compatibility can depend on Gradle/AGP/JDK/Kotlin/NDK combinations.

---

## Q113. What is Android Gradle Plugin (AGP)?

### Interview answer

The Android Gradle Plugin integrates Android-specific build logic into Gradle. It controls Android compilation, packaging, resource processing, manifest merging, variant configuration, and other platform-specific build behavior.

In React Native projects, compatibility matters between:

```text
React Native
AGP
Gradle
JDK
Kotlin
NDK
Android SDK
```

---

## Q114. What is CocoaPods?

CocoaPods is a dependency management system commonly used by iOS projects to integrate native libraries.

Typical command:

```bash
cd ios
pod install
```

You should know the difference between changing JavaScript dependencies and integrating native iOS dependencies.

---

## Q115. Why can `pod install` fail after adding a package?

Possible reasons:

- incompatible native dependency versions
- deployment target mismatch
- Xcode incompatibility
- Ruby/CocoaPods environment issues
- framework/static library settings
- architecture problems
- package-specific post-install requirements
- stale Pod state

Do not give “delete Pods and reinstall” as your entire answer. First identify the actual failing dependency or build phase.

---

## Q116. How do you troubleshoot a React Native Android build failure?

### Strong sequence

```text
1. Read the first meaningful error.
2. Identify the failing Gradle task.
3. Classify the problem:
   JS / Gradle / Kotlin / Java / C++ / CMake / NDK / manifest / dependency.
4. Check version compatibility.
5. Inspect package-specific native setup.
6. Reproduce with the smallest relevant command.
7. Clean only the affected build state.
8. Rebuild and verify.
```

Avoid blindly deleting all caches. Destructive cleanup is a troubleshooting tool, not a diagnosis.

---

## Q117. Why can a React Native library work on one RN version and fail on another?

Native packages can depend on:

- React Native APIs
- New Architecture support
- JSI
- Fabric
- TurboModules
- Gradle/AGP
- Kotlin
- NDK/CMake
- Android SDK
- iOS deployment targets
- Xcode/Swift
- autolinking

Version compatibility is effectively a matrix.

```text
RN version
   ×
package version
   ×
platform tooling
   ×
architecture mode
```

---

## Q118. What is autolinking?

### Interview answer

Autolinking allows React Native projects to automatically discover and integrate many native packages according to their package metadata/configuration instead of requiring developers to manually register every dependency.

When autolinking fails, investigate:

- package metadata
- native project configuration
- unsupported platform
- New Architecture integration
- manual linking leftovers
- build/cache state

---

## Q119. What are Android build variants and iOS schemes/configurations?

A production app often needs more than one environment.

Android commonly uses build types/flavors:

```text
Debug
Release
QA
Production
```

Organizations may combine flavors and build types.

iOS uses schemes/configurations to select environment and build settings.

A strong setup keeps environment-specific values explicit and repeatable rather than hidden in source code.

---

## Q120. How do you handle Android and iOS permissions?

Use the platform's permission model and request permissions as close as possible to the feature that needs them.

Consider:

```text
permission rationale
least privilege
first-use timing
permanently denied state
settings fallback
platform differences
privacy requirements
```

Examples include camera, microphone, location, notifications, and photo/media access.

---

# Part 11 — Testing

## Q121. What should you test in a React Native application?

Prioritize observable behavior and business risk.

Test:

- core business logic
- component behavior
- user interactions
- navigation flows
- API states
- authentication
- payment-critical flows
- edge cases
- important accessibility behavior

Do not equate a large number of snapshot files with strong test coverage.

---

## Q122. Jest vs React Native Testing Library.

### Jest

Useful for:

- unit tests
- mocking
- assertions
- pure functions
- state logic

### React Native Testing Library

Useful for:

- component behavior
- screen-level interactions
- querying rendered output
- simulating user actions

Preferred mindset:

> Test what the user can observe and do rather than overfitting tests to component implementation details.

---

## Q123. What is snapshot testing?

Snapshot testing stores a serialized representation of output and compares future output to that snapshot.

It can help with stable UI structures but becomes noisy when overused. A failing snapshot should lead to a meaningful review rather than an automatic “update snapshot” action.

---

## Q124. What should you mock?

Mock external boundaries when necessary:

- HTTP APIs
- native device services
- storage
- analytics
- time
- navigation boundaries
- platform-specific modules

Avoid mocking every internal function just to make a test green. The more internal implementation you mock, the more fragile your tests can become.

---

## Q125. Unit vs integration vs E2E.

```text
Unit
→ isolated logic

Integration
→ multiple pieces working together

E2E
→ real user flow through the application
```

A healthy suite commonly has many fast unit/component tests, fewer integration tests, and a smaller set of high-value E2E scenarios.

---

# Part 12 — Architecture and Code Organization

## Q126. How would you structure a large React Native application?

One scalable option is feature-oriented organization:

```text
src/
├── app/
│   ├── navigation/
│   ├── providers/
│   └── config/
│
├── features/
│   ├── auth/
│   │   ├── screens/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── store/
│   │   └── types/
│   ├── products/
│   ├── cart/
│   └── profile/
│
├── components/
│   ├── Button/
│   ├── Input/
│   └── Modal/
│
├── services/
│   ├── api/
│   ├── storage/
│   ├── analytics/
│   └── notifications/
│
├── utils/
├── constants/
├── types/
└── assets/
```

The exact tree is less important than:

- feature ownership
- dependency direction
- testability
- reusable primitives
- clear boundaries
- minimal coupling

---

## Q127. What is a good component architecture?

Avoid giant screen components.

Bad:

```text
ProductScreen.tsx
  → 1000+ lines
  → API calls
  → form validation
  → payment logic
  → store updates
  → analytics
  → JSX
```

Prefer:

```text
ProductScreen
 ├── ProductHeader
 ├── ProductGallery
 ├── ProductInfo
 ├── ProductVariants
 ├── ProductPrice
 ├── AddToCart
 └── RecommendationList
```

Business logic can move into hooks/services/selectors while the screen coordinates presentation and interaction.

---

## Q128. How do you decide whether something should be a component, hook, utility, or service?

### Component

UI responsibility:

```text
Button
Card
Header
ProductRow
```

### Hook

Reusable React-aware stateful logic:

```text
useAuth()
useProducts()
useNetworkStatus()
```

### Utility

Pure, generic logic:

```text
formatCurrency()
validateEmail()
calculateTotal()
```

### Service

External/system interaction:

```text
API
storage
analytics
payments
notifications
```

---

## Q129. What is separation of concerns?

The idea is to avoid making one module responsible for unrelated jobs.

Instead of:

```text
Screen
├─ API
├─ validation
├─ payment
├─ persistence
├─ analytics
├─ business rules
└─ 700 lines of JSX
```

Prefer:

```text
Screen
   ↓
Hook / ViewModel
   ↓
Domain/service logic
   ↓
API / Store / Persistence
```

The separation should make testing, reuse, and debugging easier—not simply create dozens of tiny files with no real boundaries.

---

## Q130. What is dependency inversion in a mobile application?

### Interview answer

High-level business logic should depend on abstractions rather than directly on platform/network implementations where practical.

For example:

```text
CheckoutService
      ↓
PaymentGateway interface
      ↓
StripePaymentGateway
```

Testing can inject:

```text
FakePaymentGateway
```

This is useful in apps with multiple backend/payment providers, platform-specific implementations, or heavy testing requirements.

---

## Q131. How do you prevent a project from becoming a “god screen” architecture?

Use clear boundaries:

- screen coordinates UI
- hooks manage reusable interaction/state logic
- domain services own business operations
- selectors derive state
- API layer handles transport concerns
- components remain focused

Also set team conventions early and enforce them with review, linting, and architecture documentation.

---

# Part 13 — Authentication and Security

## Q132. How would you design authentication?

Typical structure:

```text
Login
 ↓
Access Token + Refresh Token
 ↓
Secure Storage
 ↓
API Client
 ↓
401?
 ↓
Refresh
 ↓
Retry
```

Also consider:

- app startup session restoration
- token expiration
- logout
- refresh failure
- multi-device sessions
- session revocation
- biometric re-authentication where required
- secure storage
- server-side authorization

---

## Q133. What happens when the refresh token expires?

Do not keep retrying forever.

```text
API
 ↓
401
 ↓
refresh attempt
 ↓
refresh fails
 ↓
clear session
 ↓
redirect to login
```

Notify the UI appropriately and prevent multiple screens from independently showing conflicting auth states.

---

## Q134. How do you secure API communication?

At minimum:

- use HTTPS
- validate server certificates according to platform/runtime standards
- avoid logging secrets/tokens
- use secure credential storage
- enforce server-side authorization
- minimize sensitive data in the client
- avoid shipping private keys in the app bundle

For higher-risk apps, security requirements may include certificate pinning, device integrity controls, jailbreak/root detection, or other measures depending on the threat model.

Do not overpromise that any mobile client can be made completely secret or tamper-proof.

---

## Q135. Can you hide an API secret inside the React Native bundle?

No, not reliably.

### Interview answer

Any secret shipped to a client can potentially be extracted by someone who controls the device or analyzes the application package. Sensitive secrets should remain on trusted backend infrastructure.

Public application configuration can be bundled. Private server credentials cannot be made secure simply by obfuscating them.

---

# Part 14 — Production, Release, CI/CD, and OTA

## Q136. How would you create Dev / QA / Production environments?

A clean model is:

```text
                         App
                          |
                  Environment Config
              /            |            \
            DEV            QA            PROD
             |              |              |
           API-A          API-B          API-C
```

Typical controls:

- environment variables
- Expo app config
- EAS build profiles
- separate bundle/application identifiers where needed
- separate Firebase projects if required
- separate API endpoints
- environment-specific analytics
- distinct update channels/streams

Avoid scattering hardcoded production URLs across the codebase.

---

## Q137. How do you handle secrets in CI/CD?

Store secrets in the CI provider or secure build service's secret management system, not in Git.

Examples:

```text
API private key
signing credentials
service-account credentials
third-party secrets
```

Even environment variables in a client build can become public if they are bundled into the application. CI secrecy only helps for credentials that remain on the server/build side.

---

## Q138. Production crash only; development is fine. What do you investigate?

Check:

```text
release-only code paths
minification
Hermes/runtime differences
native release settings
R8/ProGuard rules
permissions
configuration/environment variables
missing assets
native package compatibility
EAS build profile
OTA update/runtime compatibility
crash logs
```

The first step should be obtaining a real crash stack/diagnostic signal rather than guessing.

---

## Q139. The app works in Expo Go but fails in a development build. Why?

Potential causes:

- a custom native dependency
- missing config-plugin changes
- native permission/configuration mismatch
- Android/iOS project configuration
- a package that is included in one client but not another
- native initialization code

Remember:

```text
Expo Go
≠
your project's custom native binary
```

---

## Q140. How do you manage release versions?

At minimum, distinguish:

```text
Human-facing version
→ 1.8.0

Platform build/version code
→ incrementing build identifier
```

A mature release process should define when each changes, how it is generated, and how CI validates that versions are unique and consistent.

---

## Q141. What is CI/CD for React Native?

### CI

Automatically validate changes:

```text
install
→ lint
→ typecheck
→ unit tests
→ build/test checks
```

### CD

Automate release steps:

```text
build
→ sign
→ distribute
→ publish/update
→ monitor
```

Common release gates include tests, environment validation, changelog/version checks, and manual approval for production.

---

## Q142. How would you implement phased rollout?

A strong mobile release strategy can use:

```text
Internal testing
 ↓
Small percentage
 ↓
Monitoring
 ↓
Expanded rollout
 ↓
100%
```

Monitor crash rate, ANR rate, API errors, startup metrics, and business-critical flows.

---

# Part 15 — Scenario-Based 5+ Years Questions

## Q143. Your screen makes an API call twice. What do you check?

### Strong answer

I would investigate systematically:

1. Is development Strict Mode contributing to repeated effect behavior?
2. Is the effect dependency array correct?
3. Is the screen mounting/unmounting unexpectedly because of navigation?
4. Is there another API invocation path?
5. Does a state update trigger an effect that calls the API again?
6. Is a parent remounting the component?
7. Is a data-fetching library already doing its own refetch?
8. Is an interceptor retrying the request?

Example loop:

```js
useEffect(() => {
  fetchData();
}, [data]);
```

If `fetchData()` updates `data`, this can create repeated requests.

---

## Q144. The app freezes when opening a screen. What do you check?

First determine whether the freeze is caused by synchronous JavaScript work, navigation/rendering, native initialization, or a network wait that is incorrectly blocking UI logic.

Look for:

- large loops
- heavy JSON transformations
- synchronous storage operations
- expensive render calculations
- huge lists
- expensive image decoding
- native module initialization
- repeated state updates

Never use a loader to hide a synchronous block of the main/JS execution path; remove or defer the blocking work.

---

## Q145. FlatList freezes while scrolling. What do you do?

Check:

```text
Row complexity
Image sizes
Row rerenders
Unstable props
Selectors
Nested lists
JS-thread work
Animation/gesture work
Virtualization configuration
```

Then profile and optimize the specific hotspot.

---

## Q146. API request takes 8 seconds. How do you optimize it?

First determine whether the 8 seconds are:

```text
client wait
network latency
server processing
payload transfer
client parsing
rendering
```

Then, depending on the cause:

- optimize backend queries
- paginate
- reduce payload
- compress responses
- cache
- parallelize independent calls
- prefetch
- render partial UI early
- defer non-critical work

---

## Q147. App crashes only after navigating back and forth many times. What do you suspect?

Potentially:

- leaked listeners
- timers
- subscriptions
- retained screen state
- WebSockets
- native resources
- accumulating caches
- repeated event handlers

Reproduce the navigation loop and inspect resource/memory growth.

---

## Q148. A third-party library breaks after a React Native upgrade. What is your process?

```text
1. Check React Native release notes.
2. Check library compatibility matrix/changelog.
3. Check New Architecture support.
4. Check native build tool versions.
5. Inspect the native compilation/runtime error.
6. Reproduce in a minimal example if necessary.
7. Upgrade/downgrade compatible versions.
8. Replace the library if it is unmaintained or incompatible.
```

Don't blindly force incompatible versions through Gradle or CocoaPods without understanding the native ABI/API implications.

---

## Q149. A library supports React Native but crashes on the New Architecture. What do you do?

Check:

- RN version
- library version
- New Architecture support status
- Fabric compatibility if it is a UI library
- TurboModule/JSI requirements if it is a native module
- codegen requirements
- Android/iOS native setup

Then reproduce with the smallest possible example and select a compatible release or alternative.

---

## Q150. How would you implement backend-driven UI?

Suppose the backend returns:

```json
{
  "home": {
    "title": "Welcome",
    "showBanner": true,
    "sections": [
      {
        "type": "banner",
        "image": "https://cdn.example.com/banner.png"
      },
      {
        "type": "productCarousel",
        "categoryId": "123"
      }
    ]
  }
}
```

A safe architecture is:

```text
Backend configuration
        ↓
Schema validation
        ↓
Normalization
        ↓
Component registry
        ↓
UI renderer
```

Example:

```js
const componentRegistry = {
  banner: Banner,
  productCarousel: ProductCarousel,
  categoryGrid: CategoryGrid,
};
```

The backend chooses among **known, validated component types**.

### Critical security point

Do not allow the backend to download and execute arbitrary JavaScript as part of normal content configuration. Prefer a declarative schema and a controlled registry.

---

## Q151. How would you build a React Native app as a SaaS white-label platform?

Use three levels of configuration.

### 1. Runtime configuration

```text
colors
text
features
API content
ordering
visibility
```

Fetched from APIs.

### 2. Build-time branding

```text
app display name
launcher icon
bundle identifier
package identifier
native splash assets
```

Produced by the build pipeline.

### 3. Tenant/backend configuration

```text
tenant ID
feature flags
content schema
branding settings
API endpoints
permissions
```

Conceptually:

```text
                 SaaS Platform
                      |
          ---------------------------
          |            |            |
       Tenant A     Tenant B     Tenant C
          |            |            |
      Config API    Config API    Config API
          |            |            |
       RN App      RN App        RN App
          |            |            |
      EAS Builds / Releases / Updates
```

For a true white-label product, the pipeline may need a separate native binary per tenant or per branding configuration, especially when icons, names, package IDs, signing credentials, or store listings differ.

---

## Q152. How would you implement feature flags?

Feature flags should be typed, centrally evaluated, and have safe defaults.

Example:

```js
const flags = {
  newCheckout: false,
  enableWallet: true,
};
```

Important concerns:

- default behavior when the flag service is unavailable
- caching
- rollout percentage
- tenant/user targeting
- auditability
- stale flags
- cleanup of old flags

Avoid allowing arbitrary remote flags to bypass authorization or security controls.

---

## Q153. How would you implement analytics without slowing the app?

Treat analytics as non-critical background work where possible.

Design:

```text
UI event
  ↓
Analytics abstraction
  ↓
queue/buffer
  ↓
batch/send
```

Consider:

- batching
- deduplication
- event sampling
- offline queue
- sensitive data filtering
- avoiding blocking the UI

Keep analytics calls out of core business logic where possible through an abstraction.

---

## Q154. How would you design push notifications?

Break the problem into:

```text
Permission
 ↓
Token registration
 ↓
Backend device-token mapping
 ↓
Notification send
 ↓
Foreground/background/terminated handling
 ↓
Deep link navigation
```

Also design token refresh and invalid-token cleanup.

---

## Q155. How would you design an app that works with poor connectivity?

Use:

```text
Local cache
 ↓
Network awareness
 ↓
Optimistic UI where safe
 ↓
Retry queue
 ↓
Conflict resolution
```

Do not blindly replay non-idempotent operations. Use server-supported idempotency keys where necessary.

---

# Part 16 — Coding Questions

## Coding 1. Reverse a string.

```js
function reverse(str) {
  return [...str].reverse().join('');
}
```

### Follow-up

Why use spread rather than `str.split('')`?

A discussion can cover Unicode code points versus UTF-16 code units. For true user-perceived grapheme reversal, code points alone are still not always sufficient.

---

## Coding 2. Remove duplicates.

```js
const unique = [...new Set(arr)];
```

### Follow-up

If objects are involved, `Set` deduplicates object references, not objects by content. For value-based deduplication you need a key or normalization strategy.

---

## Coding 3. Group an array by property.

```js
const result = users.reduce((acc, user) => {
  const key = user.role;

  if (!acc[key]) {
    acc[key] = [];
  }

  acc[key].push(user);
  return acc;
}, {});
```

Possible follow-up:

> How would you avoid prototype-key collisions?

Possible answer: use `Object.create(null)`, `Map`, or an explicitly validated key strategy depending on requirements.

---

## Coding 4. Implement debounce.

```js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

Understand:

- closure
- timer replacement
- preserving `this`
- forwarding arguments
- cleanup/cancel support in more complete implementations

### Senior follow-up

How would you add:

```text
cancel()
flush()
leading execution
trailing execution
```

---

## Coding 5. Flatten an array.

Built-in:

```js
const result = arr.flat(Infinity);
```

Manual recursive version:

```js
function flatten(arr) {
  const result = [];

  for (const item of arr) {
    if (Array.isArray(item)) {
      result.push(...flatten(item));
    } else {
      result.push(item);
    }
  }

  return result;
}
```

### Senior discussion

Be ready to discuss recursion depth, iterative implementations, time/space complexity, and whether sparse arrays/typed arrays need special handling.

---

## Coding 6. Find the first non-repeating character.

```js
function firstUniqueChar(str) {
  const counts = new Map();

  for (const char of str) {
    counts.set(char, (counts.get(char) ?? 0) + 1);
  }

  for (const char of str) {
    if (counts.get(char) === 1) {
      return char;
    }
  }

  return null;
}
```

Complexity is approximately:

```text
Time  → O(n)
Space → O(k)
```

where `k` is the number of unique characters tracked.

---

## Coding 7. Implement memoization.

```js
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

### Senior caveat

`JSON.stringify(args)` is not a universal cache-key strategy. It can be expensive, order-sensitive for some objects, and unable to represent all values. A production implementation must choose a keying/equality strategy appropriate to the inputs.

---

## Coding 8. Implement a promise concurrency limiter.

A common 5+ year coding question is:

> Given 100 API tasks, run at most 5 at a time.

Mental model:

```text
100 tasks
   ↓
5 active
5 active
5 active
...
```

You should be able to implement a worker-pool or queue-based solution and explain:

- concurrency limit
- result ordering
- failure handling
- cancellation
- backpressure

---

## Coding 9. Implement retry with exponential backoff.

Concept:

```text
Attempt 1 → immediate / small delay
Attempt 2 → larger delay
Attempt 3 → larger delay
...
```

Typical formula:

```js
delay = baseDelay * 2 ** attempt;
```

Production concerns:

- maximum retries
- maximum delay
- jitter
- only retry transient/idempotent failures
- cancellation
- authentication failures
- server-provided retry hints

---

# Part 17 — Rapid-Fire Questions

## Q156. `null` vs `undefined`

`undefined` generally means a value is missing/unassigned, while `null` usually represents an intentional absence of a value.

---

## Q157. `map` vs `forEach`

```text
map
→ transforms and returns a new array

forEach
→ executes a callback for side effects and returns undefined
```

---

## Q158. `filter` vs `find`

```text
filter
→ all matching elements

find
→ first matching element
```

---

## Q159. `some` vs `every`

```text
some
→ at least one item satisfies condition

every
→ all items satisfy condition
```

---

## Q160. Why is `NaN !== NaN`?

Because JavaScript defines `NaN` as not equal to itself under ordinary equality semantics.

Use:

```js
Number.isNaN(value)
```

for an explicit NaN check.

---

## Q161. What is optional chaining?

```js
user?.profile?.address?.city
```

It stops the property access chain when an intermediate value is `null` or `undefined` and returns `undefined` rather than throwing for that chain.

---

## Q162. What is nullish coalescing?

```js
value ?? defaultValue
```

Uses the default only when the left side is `null` or `undefined`.

Compare:

```js
value || defaultValue
```

which also treats values such as `0`, `false`, and `''` as falsy.

---

## Q163. What is destructuring?

```js
const { name, age } = user;
```

Array destructuring:

```js
const [first, second] = items;
```

---

## Q164. What are default parameters?

```js
function greet(name = 'Guest') {}
```

The default applies when the argument is `undefined`.

---

## Q165. What is currying?

Currying transforms a multi-argument function conceptually like:

```js
add(a, b)
```

into:

```js
add(a)(b)
```

Example:

```js
const add = a => b => a + b;
```

---

## Q166. What is a pure function?

A pure function:

- returns the same output for the same input
- has no observable side effects
- does not mutate external state

Example:

```js
const add = (a, b) => a + b;
```

Pure logic is easier to test, cache, and reason about.

---

## Q167. What is referential equality?

Two separate objects with the same content are still different references:

```js
{} === {}; // false
```

Reference identity is important for React memoization and many state-management optimizations.

---

## Q168. Why can `useSelector` cause rerenders?

If the selected value changes according to the equality logic being used, the component can rerender.

A problematic selector may create a fresh object every time:

```js
const selected = useSelector(state => ({
  user: state.user,
  cart: state.cart,
}));
```

Depending on the selector/equality setup, this can cause unnecessary updates. Memoized selectors or selecting stable individual values can help.

---

# Part 18 — Senior-Level Discussion Questions

## Q169. Why use Redux?

Weak answer:

> Redux stores global state.

Strong answer:

> I introduce Redux when state is sufficiently shared or complex that centralized, predictable transitions and tooling provide value. I consider data ownership first and avoid putting transient component state or every server response into a global store.

---

## Q170. Why use `useMemo`?

Weak answer:

> To improve performance.

Strong answer:

> `useMemo` trades memoization overhead and dependency tracking for reusing a calculated value. I use it when the calculation is expensive or when stable reference identity is important to downstream rendering. I don't treat memoization as a blanket best practice.

---

## Q171. Why use `useCallback`?

Weak answer:

> It makes functions faster.

Strong answer:

> It stabilizes a function reference across renders until dependencies change. That matters when referential identity influences memoized child components or dependency tracking. It does not inherently make the function execute faster.

---

## Q172. Why use Expo?

Weak answer:

> Expo makes React Native easier.

Strong answer:

> Expo provides standardized development tooling, native modules, builds, updates, and deployment workflows while still allowing native customization through development builds, prebuild, config plugins, and custom native modules.

---

## Q173. Why might you choose React Native CLI-style native control instead of a highly managed Expo workflow?

### Strong answer

I would choose the workflow based on native requirements and operational constraints, not on whether one approach is inherently more professional. If the product depends on specialized native integrations, a tightly controlled native build environment, or existing native applications, more direct native control may be useful. If the product benefits from Expo's tooling and supported modules, Expo can reduce maintenance overhead.

---

## Q174. What is more important: code reuse or native performance?

Neither is universally more important.

A strong answer is:

> I optimize for product requirements. Shared business logic is valuable, but forcing every platform into identical UI or behavior can create unnecessary compromises. I prefer shared domain logic and components where the UX should be consistent, while allowing platform-specific implementations when they improve correctness or user experience.

---

## Q175. How do you decide if a third-party library should be added?

Evaluate:

```text
Maintenance activity
Compatibility with current RN/Expo
New Architecture support
Bundle/native cost
License
Security history
Issue quality
API quality
Community/adoption
Ability to replace/remove later
```

A tiny package with poor maintenance can create larger costs than writing a small internal utility.

---

## Q176. When should business logic live on the backend rather than in the app?

Keep security-sensitive and authoritative rules on the server.

For example:

```text
Price authority
Authorization
Entitlement
Payment finalization
Fraud checks
Sensitive business policy
```

The client can provide UX validation and calculations, but the server should remain authoritative for security-sensitive decisions.

---

## Q177. What is server state vs client state?

### Client state

Owned primarily by the application:

```text
modal visibility
selected tab
form draft
local preferences
UI state
```

### Server state

Owned by backend systems:

```text
products
orders
profile data
inventory
notifications
```

Server state has unique concerns:

```text
cache
freshness
refetch
staleness
invalidation
pagination
synchronization
```

This is why a dedicated data-fetching/cache layer can be preferable to treating every API response as ordinary global UI state.

---

## Q178. How would you design a reusable payment flow?

Use a domain-level payment abstraction:

```text
Checkout UI
   ↓
Checkout service
   ↓
Payment gateway interface
   ↓
Stripe / another provider
```

The client should not be the authority for payment success. After the payment flow, the application should verify server-side order/payment state.

Also consider:

- duplicate taps
- idempotency
- timeout/retry
- cancellation
- app backgrounding
- deep links
- partial failures
- analytics

---

## Q179. How would you debug a production issue you cannot reproduce locally?

Use observability:

```text
Crash report
 ↓
Stack trace
 ↓
Release/version/build
 ↓
Device/OS
 ↓
User/session context (privacy-safe)
 ↓
API/network logs
 ↓
Recent deployment/change
```

Then narrow the condition rather than guessing.

A mature team should be able to correlate:

```text
app version
build number
environment
release/OTA update
backend version
```

---

## Q180. How do you evaluate whether a performance optimization actually worked?

Before optimization define a measurable metric.

Examples:

```text
startup time
screen transition time
JS frame workload
UI frame drops
render count
memory baseline
API latency
list scroll stability
```

Then compare the same workload before and after the change.

A performance change that “feels faster” but cannot be measured is less convincing in a senior review.

---

# Final Revision Checklist

## JavaScript

- [ ] Scope
- [ ] Hoisting
- [ ] Temporal Dead Zone
- [ ] Closures
- [ ] Stale closures
- [ ] `this`
- [ ] `call`, `apply`, `bind`
- [ ] Prototype chain
- [ ] Classes
- [ ] Shallow/deep copy
- [ ] Immutability
- [ ] Equality
- [ ] Call stack
- [ ] Event loop
- [ ] Microtasks/tasks
- [ ] Promises
- [ ] `async/await`
- [ ] Debounce/throttle
- [ ] Garbage collection
- [ ] Memory leaks

## React

- [ ] Components
- [ ] Props/state
- [ ] Render purity
- [ ] Render phase / commit phase
- [ ] Reconciliation
- [ ] Keys
- [ ] State batching
- [ ] Functional updates
- [ ] `useState`
- [ ] `useEffect`
- [ ] Effect dependencies
- [ ] Effect cleanup
- [ ] `useMemo`
- [ ] `useCallback`
- [ ] `React.memo`
- [ ] `useRef`
- [ ] Context
- [ ] Custom hooks
- [ ] Rules of Hooks
- [ ] Strict Mode
- [ ] Suspense/lazy loading
- [ ] Error boundaries

## React Native

- [ ] RN architecture
- [ ] Legacy Bridge
- [ ] JSI
- [ ] Fabric
- [ ] TurboModules
- [ ] Hermes
- [ ] JS/UI/native execution concepts
- [ ] FlatList
- [ ] SectionList
- [ ] VirtualizedList
- [ ] `getItemLayout`
- [ ] list optimization
- [ ] images
- [ ] startup
- [ ] animations
- [ ] gestures
- [ ] native modules
- [ ] autolinking

## Expo

- [ ] Expo Go
- [ ] Development builds
- [ ] Prebuild
- [ ] Config plugins
- [ ] EAS Build
- [ ] EAS Submit
- [ ] EAS Update
- [ ] Build profiles
- [ ] Runtime version compatibility
- [ ] Environment variables
- [ ] Expo Router
- [ ] Deep linking
- [ ] Branding/build-time configuration

## State management

- [ ] Context
- [ ] Redux Toolkit
- [ ] Immer
- [ ] Middleware
- [ ] Selectors
- [ ] Memoized selectors
- [ ] RTK Query
- [ ] Client state vs server state
- [ ] Persistence

## Networking

- [ ] API layer
- [ ] Axios/fetch abstraction
- [ ] Interceptors
- [ ] Access token
- [ ] Refresh token
- [ ] Refresh race conditions
- [ ] Cancellation
- [ ] Retry/backoff
- [ ] Pagination
- [ ] Caching
- [ ] Error states

## Native / production

- [ ] Gradle
- [ ] AGP
- [ ] Kotlin/Java
- [ ] NDK/CMake basics
- [ ] CocoaPods
- [ ] Xcode
- [ ] Permissions
- [ ] Release configuration
- [ ] CI/CD
- [ ] Crash reporting
- [ ] OTA/runtime compatibility
- [ ] Store release lifecycle

## Senior scenarios

- [ ] API called twice
- [ ] FlatList lag
- [ ] app freeze
- [ ] startup slowdown
- [ ] production-only crash
- [ ] Expo Go vs dev build mismatch
- [ ] incompatible third-party library
- [ ] authentication race
- [ ] offline support
- [ ] feature flags
- [ ] backend-driven UI
- [ ] white-label SaaS architecture
- [ ] payment architecture
- [ ] observability

---

# Recommended 5+ Years Mock Interview Sequence

Use this order for repeated practice:

```text
Round 1 — JavaScript fundamentals
        ↓
Round 2 — React rendering + hooks
        ↓
Round 3 — React Native architecture
        ↓
Round 4 — Expo + EAS
        ↓
Round 5 — Performance/debugging
        ↓
Round 6 — State/networking/auth
        ↓
Round 7 — Native Android/iOS
        ↓
Round 8 — Architecture/system design
        ↓
Round 9 — Coding round
        ↓
Round 10 — Project deep dive
```

For every round, practice answering each question in:

```text
20–30 seconds → concise interview answer
60–90 seconds → detailed explanation
3–5 minutes   → architecture/troubleshooting discussion
```

---

# Project Deep-Dive Questions You Should Prepare

Interviewers often move from theory into your actual experience. Prepare strong, truthful answers for:

1. Walk me through your most complex React Native application.
2. Why did you choose React Native/Expo?
3. Why did you choose your state-management approach?
4. How did you handle authentication?
5. How did you solve token refresh?
6. How did you optimize a slow FlatList?
7. What was your toughest Android build issue?
8. What was your toughest iOS issue?
9. Did you work with native modules?
10. How did you manage environments?
11. How did you release builds?
12. How did you troubleshoot a production crash?
13. What architecture did you use and why?
14. How did you prevent duplicate API requests?
15. How did you handle offline/error states?
16. How did you implement payments?
17. How did you handle push notifications?
18. What was your biggest performance problem?
19. What technical decision would you change if rebuilding the project?
20. Tell me about a production incident and how you fixed it.

**Important:** only claim technologies, scale, architecture decisions, performance improvements, native work, or production ownership that you can actually explain in depth.

---

# How to Sound Like a 5+ Years Candidate

Avoid absolute statements such as:

```text
“Redux is always better.”
“useMemo should always be used.”
“Expo cannot use native code.”
“await blocks JavaScript.”
“FlatList makes everything fast.”
“AsyncStorage is secure.”
“Memoization prevents rerenders.”
```

Prefer contextual answers:

```text
“It depends on the ownership and update frequency of the state.”
“I would measure before optimizing.”
“For native customization, I would use a development build/prebuild/config plugin or native project code.”
“await suspends the async function's continuation rather than blocking the entire runtime.”
“FlatList provides virtualization, but row complexity and JS/UI workload still matter.”
“For sensitive data I would use platform-backed secure storage.”
“Memoization can skip work when references/values are stable; it is not a universal rerender switch.”
```

That style communicates engineering judgment rather than memorization.

---

# Current Official References

For current React Native/Expo interview preparation, prioritize official documentation because architecture and tooling evolve.

- React Native 0.82 / New Architecture: https://reactnative.dev/blog/2025/10/08/react-native-0.82
- Expo New Architecture guide: https://docs.expo.dev/guides/new-architecture/
- Expo development builds: https://docs.expo.dev/develop/development-builds/introduction/
- Expo prebuild / CNG: https://docs.expo.dev/workflow/prebuild/
- Expo Config Plugins: https://docs.expo.dev/config-plugins/introduction/
- EAS Build: https://docs.expo.dev/build/introduction/
- EAS Update / runtime compatibility: https://docs.expo.dev/build/updates/
- Expo Router: https://docs.expo.dev/router/introduction/
- React Native performance overview: https://reactnative.dev/docs/performance
- React Native FlatList: https://reactnative.dev/docs/flatlist
- React Native VirtualizedList: https://reactnative.dev/docs/virtualizedlist

> **Current-tooling note:** React Native and Expo release frequently. Before an interview, verify the latest versions and official compatibility guidance for the exact stack the company uses.

---

# End Goal

By the time you finish this guide, you should be able to move naturally through:

```text
JavaScript
   ↓
React
   ↓
React Native
   ↓
Expo
   ↓
Performance
   ↓
Architecture
   ↓
Native debugging
   ↓
Production systems
```

The target is not to memorize 180 answers. The target is to build a mental model strong enough that you can explain **why** the code works, **when** to use a technology, **what trade-offs** it introduces, and **how you would debug it in production**.
