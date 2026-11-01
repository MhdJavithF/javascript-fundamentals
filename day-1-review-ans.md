# Day 1 Review — JavaScript Scope, Hoisting & Closures

First, I want to appreciate one thing: you pushed your first attempt to GitHub before asking for the solution.

That's exactly the habit we want to build.

Also, the fact that you initially returned `num++`, encountered a TypeError, investigated the reason, and changed your implementation is good problem-solving behaviour.

You didn't immediately ask AI to fix it. That's progress.

Let's review this as a product-company interviewer would.

## 1. Exercise 1 — Scope and hoisting

Correct — 3/3

Your answer:

```
undefined
undefined
20
10
```

All four outputs are correct.

Your explanation is also correct. One small terminology improvement:

You said:

> The binding of the variable will check the closest declaration first in block scope then functional and global scope.

More precisely:

JavaScript resolves a variable through its lexical scope chain, starting with the current scope and moving outward.

In this example:

```
var a = 10;

function test() {
  console.log(a);

  var a = 20;

  console.log(a);
}
```

The local `var a` declaration is function-scoped.

Even though its declaration appears after the first `console.log`, its binding is established when the function execution context is initialized.

Therefore, the first `console.log(a)` accesses the local `a`, which is initially `undefined`.

It does not fall back to the global `a`.

Interview-quality explanation:

"The local variable `a` is function-scoped because it is declared using `var` inside `test()`. Its binding is initialized to `undefined` before execution reaches the declaration. Therefore, the first log accesses the local binding rather than the global variable."

That's a strong explanation.

## 2. Exercise 2 — Temporal Dead Zone

Partially correct — 2.5/3

Your understanding of the TDZ is good.

You correctly identified:

- The first log throws a `ReferenceError`.
- The second log would print `5` if execution reached it.
- `let` has a TDZ.

But there's one important execution detail you missed.

```
function example() {
  console.log(count); // ReferenceError

  let count = 5;

  console.log(count);
}

example();
```

Does the second console.log execute? No.

When the first statement throws an uncaught `ReferenceError`, execution of that function stops immediately.

JavaScript doesn't skip the error and continue to the next statement.

If the error isn't caught, it propagates to the caller.

### Important correction

You said:

> For let/const the declaration will not hoist to top.

This is technically incorrect.

`let` and `const` bindings are established when their lexical environment is created. However, they remain uninitialized until execution reaches the declaration.

That uninitialized period is the Temporal Dead Zone.

Remember:

| Declaration          | Hoisted? | Initial state             |
| -------------------- | -------- | ------------------------- |
| `var`                | Yes      | `undefined`               |
| `let`                | Yes      | Uninitialized             |
| `const`              | Yes      | Uninitialized             |
| Function declaration | Yes      | Function object available |

Interview-quality explanation:

"`let` is also hoisted, but unlike `var`, it isn't initialized during environment creation. Accessing it before its declaration executes results in a ReferenceError because the binding is in the Temporal Dead Zone."

This distinction is frequently tested in JavaScript interviews.

## 3. Exercise 3 — Lexical scope vs caller scope

Correct output — explanation needs refinement

Your predicted output is correct:

```
global
```

Your explanation:

> Even though second is called inside first(), the binding of second() is bound with global object scope.

You're thinking in the right direction, but let's make it technically accurate.

The important rule is:

A function's lexical scope is determined by where the function is defined, not where it is called.

Look at this:

```
const value = "global";

function first() {
  const value = "first";
  second();
}

function second() {
  console.log(value);
}

first();
```

When `second()` is defined, it is defined in the outer/global lexical environment.

When `first()` calls `second()`, JavaScript doesn't change the lexical environment of `second()`.

So the lookup happens like this:

Variable lookup inside second()

1\. second() local scope

No local declaration of `value`.

2\. Outer lexical scope of second()

`value = "global"`

Found

The local `value = "first"` inside first() is not in second()'s lexical scope chain.

One correction: avoid saying "global object scope" as a general explanation. In modern JavaScript, global lexical bindings and properties of the global object aren't always the same thing.

Interview-quality explanation:

"JavaScript follows lexical scoping. The scope chain is determined by where a function is declared, not where it is invoked. Since `second()` is declared in the global lexical environment, it resolves `value` from that environment, even though it is called inside `first()`."

Good progress here.

## 4. Exercise 4 — Closure-based counter

Correct implementation

Your implementation:

```
function createCounter(num) {
  let start = num;

  return function () {
    return ++start;
  };
}
```

This is correct. It satisfies all the requirements.

You have independently implemented a closure.

But you mentioned something important:

> How is it remembering? I'm still a bit confused.

Let's understand this properly. This is one of the most important concepts in JavaScript.

### 4.1 First understand what happens during a function call

Imagine:

```
const counterA = createCounter(0);
```

When this executes:

1. `createCounter(0)` is called.
2. A local variable `num` receives `0`.
3. `start` is initialized to `0`.
4. An inner function is created.
5. That inner function is returned and assigned to `counterA`.

Now something interesting happens.

Normally, you might expect local variables to become inaccessible after the outer function finishes.

But here, the returned function still needs access to `start`.

JavaScript preserves the necessary lexical environment because the returned function references that variable.

This is called a closure.

### 4.2 Visualize what JavaScript remembers

Step 1 — Create counterA

createCounter(0)

# start = 0

Preserved lexical environment

`counterA` → returned function

Step 2 — Call counterA()

`start = 1`

The returned function accesses and updates the same variable.

Step 3 — Call counterA() again

`start = 2`

It doesn't recreate start. It updates the existing binding.

### 4.3 Your question about counterB

You asked:

> We aren't passing any number inside counterB(previous) like this. How does it remember the previous value properly?

Let's examine:

```
const counterA = createCounter(0);
const counterB = createCounter(10);
```

These are two separate function calls.

Each call creates its own lexical environment.

counterA

# 0

Initial start

1 → 2 → 3 → 4

counterB

# 10

Initial start

11 → 12

Two independent function instances, two independent preserved lexical environments.

Notice:

- `counterA()` accesses the environment created by `createCounter(0)`.
- `counterB()` accesses the environment created by `createCounter(10)`.
- Neither function needs another argument because its initial value was already supplied when its outer function was called.

This is why each counter maintains independent state.

### 4.4 Why your first attempt failed

Your initial implementation was:

```
function createCounter(num) {
  let start = num;
  return num++;
}
```

The issue is that `num++` returns a number, not a function.

But the requirement is to return another function.

Therefore, when you tried:

```
counterA();
```

JavaScript effectively tried to call a number as a function, resulting in:

```
TypeError: counterA is not a function
```

Your correction was exactly what the problem required.

### 4.5 One more technical detail

A closure doesn't simply take a snapshot of a variable's value.

It retains access to the variable's lexical binding.

That's why changes to `start` are visible on subsequent calls.

Interview-quality explanation:

"A closure is formed when an inner function retains access to variables from its outer lexical environment, even after the outer function has completed execution. In my counter implementation, each call to `createCounter` creates an independent lexical environment containing `start`. The returned function retains access to that binding and updates it on every invocation."

That's the explanation I want you to be able to give without memorizing it.

## 5. Senior-level scenario — Debugging unexpected values in React

Needs more practical understanding

Your answers show that you know about breakpoints and functional state updates, but you haven't yet connected them into a debugging strategy.

Let's build that skill.

### Step 1. Inspect the execution path

Your suggestion about the browser Sources tab is correct.

For example, suppose clicking a button produces an unexpected value.

I'd investigate:

1. Open Chrome DevTools.
2. Go to Sources.
3. Identify the event handler.
4. Add a breakpoint inside the handler.
5. Reproduce the issue using the same interaction sequence.
6. Inspect local variables, closure variables, and the call stack.

### Step 2. Identify which variable binding is being accessed

This is where your Day 1 learning becomes useful.

When the debugger pauses:

- Inspect the current local scope.
- Inspect the Closure section in the debugger's Scope panel, when available.
- Check the value of the variable.
- Trace where the function was defined.
- Check whether the variable is being captured from an earlier function invocation.

Chrome DevTools can show closure-related variables in its Scope panel.

This helps distinguish the variable's actual binding from what you assumed the function would access.

### Step 3. Understand the difference between a closure and a stale React state value

This is an important distinction.

A closure is a normal JavaScript feature. It isn't inherently a bug.

A stale closure occurs when a function retains access to values from an earlier render, but your application expects it to use newer values.

Consider this React example:

```
import { useEffect, useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);
      setCount(count + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <h1>{count}</h1>;
}
```

What is wrong?

The effect runs only once because its dependency array is empty.

The interval callback closes over the initial render's `count`, which is `0`.

Consequently, every interval callback reads that same value.

The state update repeatedly requests `1`, rather than incrementing the latest value.

A possible fix:

```
setCount((prevCount) => prevCount + 1);
```

This functional updater calculates the next state using the latest pending state.

However, notice something important:

```
console.log(count);
```

The functional updater doesn't automatically make this separate closure read the latest `count`.

If you need the latest value in the interval callback itself, you must address that separately.

For example, you can use an effect with the appropriate dependencies or a ref, depending on the requirements.

### Step 4. Verify the fix

After fixing the issue:

- Reproduce the original sequence of interactions.
- Check whether the correct value is being accessed.
- Verify repeated clicks and rapid interactions.
- Check cleanup of timers or event listeners.
- Test other scenarios that might introduce stale values.
- Add a regression test.

### What a strong senior-level interview answer sounds like

Here's a model answer. Understand the reasoning rather than memorizing the wording.

> First, I would reproduce the issue consistently and identify the function responsible for reading the unexpected value.
>
> I would use Chrome DevTools breakpoints to inspect the call stack, local variables, and closure scope to understand which binding the function is accessing.
>
> If the issue occurs inside a React component, I would also inspect how state is updated and whether the callback was created during an earlier render.
>
> I would distinguish between a normal JavaScript closure and a stale React closure by checking whether the callback retains an older render's state or props.
>
> Depending on the root cause, I might use a functional state updater, correct the effect dependencies, or use a ref if the callback needs access to a mutable latest value without triggering a render.
>
> Finally, I would reproduce the original issue after the fix, test repeated interactions, and add a regression test to prevent it from happening again.

This is the type of structured thinking expected in experienced frontend interviews.

# 6. Day 1 final evaluation

| Topic                  | Score | Assessment                                 |
| ---------------------- | ----- | ------------------------------------------ |
| Scope and hoisting     | 5/5   | Strong                                     |
| Temporal Dead Zone     | 3.5/5 | Needs precision                            |
| Lexical scoping        | 4/5   | Correct understanding; improve terminology |
| Closure implementation | 5/5   | Independently solved                       |
| Closure internals      | 3/5   | Concept needs reinforcement                |
| React debugging        | 2/5   | Needs practical experience                 |

Day 1 assessment

# 3.8 / 5

Good starting foundation

You can apply basic JavaScript concepts and independently solve a simple closure problem. Your biggest improvement area is explaining internal execution behaviour precisely and connecting JavaScript fundamentals to React.

## 7. Before we move to Day 2 — one final challenge

I don't want to move forward until you've demonstrated that you understand what a closure actually preserves.

Answer this without running the code:

```
function createFunctions() {
  let functions = [];

  for (var i = 0; i < 3; i++) {
    functions.push(function () {
      return i;
    });
  }

  return functions;
}

const result = createFunctions();

console.log(result[0]());
console.log(result[1]());
console.log(result[2]());
```

Questions:

1. Predict all three outputs.
2. Why do you think they produce those values?
3. How would changing `var` to `let` affect the result?
4. What exactly is being remembered by each returned function?

Don't change the code yet. Explain your reasoning.

## GitHub progress — Day 1

Attempt completed

| Activity                 | Status              |
| ------------------------ | ------------------- |
| Scope and hoisting       | Completed           |
| TDZ                      | Completed           |
| Lexical scope            | Completed           |
| Counter implementation   | Completed           |
| Closure explanation      | Needs reinforcement |
| React debugging scenario | Reviewed            |
| Final closure challenge  | Pending             |

Next topic: JavaScript Closures — deeper understanding, practical applications, and interview problems.

One thing I want you to remember throughout this preparation: