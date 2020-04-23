# RXJS, Event-based programming and Observables

Learn the power of event-based programming using Observables to work with asynchronous code.

**Recommended reading**

- [JavaScript](../js/)
- [JavaScript Closures](../js-closures/)
- [JavaScript Promises](../js-promises/)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [What is a promise?](#what-is-a-promise)
  - [Asynchronous callback styles](#asynchronous-callback-styles)
  - [Promises/A+ specification](#promisesa-specification)
  - [English, please?](#english-please)
  - [Code, please?](#code-please)
  - [Consuming a promise](#consuming-a-promise)
    - [Let's try it](#lets-try-it)
- [Basic promise behavior](#basic-promise-behavior)
  - [Promise callback syntax](#promise-callback-syntax)
    - [More promise callback syntax](#more-promise-callback-syntax)
  - [Promise callbacks are **optional**](#promise-callbacks-are-optional)
    - [Unhandled promise rejections](#unhandled-promise-rejections)
  - [Using `catch(...)`](#using-catch)
  - [Asynchronicity](#asynchronicity)
  - [Promises can only be resolved or rejected **once**](#promises-can-only-be-resolved-or-rejected-once)
  - [The result of a promise can be retrieved **later**](#the-result-of-a-promise-can-be-retrieved-later)
  - [Promises are resolved with **one** value](#promises-are-resolved-with-one-value)
  - [Promise utilities](#promise-utilities)
- [Chaining promises](#chaining-promises)
  - [Chaining `.then()` calls](#chaining-then-calls)
  - [Promise resolution procedure](#promise-resolution-procedure)
    - [Returning a value from the resolution or rejection callback (1 & 4)](#returning-a-value-from-the-resolution-or-rejection-callback-1--4)
    - [Throwing an error from the resolution or rejection callback (2 & 5)](#throwing-an-error-from-the-resolution-or-rejection-callback-2--5)
    - [Returning a promise in the resolution callback (3 & 6)](#returning-a-promise-in-the-resolution-callback-3--6)
    - [Returning a rejected promise in the resolution callback (3 & 6)](#returning-a-rejected-promise-in-the-resolution-callback-3--6)
  - [An example](#an-example)
  - [Resolving promises in chains](#resolving-promises-in-chains)
  - [Rejecting promises in chains](#rejecting-promises-in-chains)
  - [Behavior of a promise chain](#behavior-of-a-promise-chain)
    - [All's right with the world](#alls-right-with-the-world)
    - [Catching errors in a promise chain](#catching-errors-in-a-promise-chain)
    - [Early errors in a promise chain](#early-errors-in-a-promise-chain)
  - [Handling errors in a promise chain](#handling-errors-in-a-promise-chain)
    - [Handling error example](#handling-error-example)
    - [Handling error result](#handling-error-result)
  - [An asynchronous example](#an-asynchronous-example)
    - [An asynchronous example with error handling](#an-asynchronous-example-with-error-handling)
- [Why use promises?](#why-use-promises)
  - [Triumph over the callback hell](#triumph-over-the-callback-hell)
    - [Callback hell example](#callback-hell-example)
    - [Flatten the pyramid of doom](#flatten-the-pyramid-of-doom)
    - [Flatten the pyramid of doom with promises](#flatten-the-pyramid-of-doom-with-promises)
  - [Complex chains](#complex-chains)
- [Parallel execution](#parallel-execution)
  - [Successful parallel execution](#successful-parallel-execution)
  - [Failed parallel execution](#failed-parallel-execution)
- [`async`/`await`](#asyncawait)
  - [The problem with promises](#the-problem-with-promises)
  - [Async functions and the `await` operator](#async-functions-and-the-await-operator)
  - [`await` and rejected promises](#await-and-rejected-promises)
    - [Handling rejected promises with `await`](#handling-rejected-promises-with-await)
  - [Awaiting the result of parallel executions](#awaiting-the-result-of-parallel-executions)
  - [Async functions always return promises](#async-functions-always-return-promises)
- [Resources](#resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Setup

To follow along the examples of this subject, here is what you need to do:

1. Create a new directory in your `dfa-course` main directory. Let's call it `rxjs`
1. Create those files in this new `rxjs` directory:
  - An `index.html` file with [minimal markup][htmlshell],
  - A blank `script.js`
3. Include the **RxJS library** and the `script.js` file in `index.html` before the `&lt;/body>` tag:
  ```html
  <body>
  * <script src="https://unpkg.com/@reactivex/rxjs@6.5.5/dist/global/rxjs.umd.js"></script>
  * <script src="./script.js"></script>
  &lt;/body>
  ```

## What (the hell) is...

<h4>... an observable?</h4>

An observable is an **asynchronous data stream**, meaning that it emits **multiple events over time**.

For example, a user's **mouse clicks** on a website could easily be modeled as an observable:
there will be several of them and they will happen at different times in the future.

<h4>... reactive programming</h4>

Basically, **reactive programming** is programming with **asynchronous** data **streams** by reacting to their events.

<h4>... RxJS</h4>

**RxJS** stands for [**R**eactive**X**][rx] for **J**ava**S**cript, and is a JS library that provides an amazing toolbox of functions to **create, combine and filter** observables.

## Observing Observables

As said previously, an Observable is a stream of data. It can:

- **Emit different values** over time
- **Emit an error** event and stop
- **Complete** gracefully and stop

Many documentation explaining Observables and reactive programming do so by using **marble diagrams** to represent them graphically. Here's one that resumes the three properties above (using a "click" stream example):

<p class="center shadow"><img src="images/reactive-programming.png"></p>

## Simple illustration

Suppose you want to react to clicks on a particular button in your page:

```html
<body>
  <button id="myButton">Button</button>

  <!-- Scripts -->
&lt;/body>
```

This could be achieved by adding an `EventListener` to said button and providing a callback:

```js
document
  .getElementById("myButton")
  .addEventListener(
    "click",
    () => alert("Button has been clicked")
  );
```

### Work with Observables

To do the same thing using `Observable` and the **RxJS** library is not that much complicated.

Remove any previous code in `script.js` and add:

```js
const { fromEvent } = rxjs; // `1`

fromEvent(document.getElementById("myButton"), "click") // `2`
  .subscribe(() => alert("Button has been clicked")); // `3`
```
**`1`** - With the CDN script included, the **RxJS** library is globally available through the `rxjs` variable. We used here some [destructuring assignment][destructuring] to get the `fromEvent(...)` function.

**`2`** - [`fromEvent(...)`][from-event] creates an `Observable` from **a particular event** occuring on **a particular DOM element**. In this case, an `Obervable` for `click` events on the `#myButton` element.

**`3`** - `Observable`s won't **emit** or even **do** anything until an **Observer subscribes to them** by calling their `subscribe(...)` method, and passing it a callback. For each emitted value, this callback is called with the value as argument.

> Go on and click the button, now.

### Complicate all the things

Suppose now, for the sake of the argument, that we want to react to the click button only if the user **pressed the `Shift` key** while clicking, in which case we display in an alert **the coordinates of the mouse**.

Without using **RxJS** and `Observables`, we could write something like this:

```js
document.getElementById("myButton").addEventListener("click", (event) => {
  // Check if the event has been fired while the Shift key was pressed
* if (event.shiftKey) {
    // Extract the mouse coordinates from the MouseEvent object
*   const coordinates = {
*     x: event.pageX,
*     y: event.pageY
*   };
    // Display the coordinates in an alert string
*   alert(\`Clicked at [${coordinates.x}, ${coordinates.y}]`);
* }
});
```
At first glance, nothing prevents us to write **the exact same code** in an `Observable`'s `subscribe` callback and  go on our merry way.

> That would work, but **RxJS** allows you to be more structured..

#### The RxJs way!

Here is how you would write this feature, following the **RxJS** philosophy:

```js
const { fromEvent } = rxjs;
const { filter, map } = rxjs.operators;

fromEvent(document.getElementById("myButton"), "click")
  .pipe(
    filter((click) => click.shiftKey),
    map((event) => ({ x: event.pageX, y: event.pageY }))
  )
  .subscribe((coordinates) =>
    alert(\`Clicked at [${coordinates.x}, ${coordinates.y}]`)
  );
```
Hum. Okay.

That's **a lot of new code**.

Let's dive into it before discussing what advantages this implementation offers over the previous one.

> So... what is this `pipe(...)` method anyway ?

## Introducing... Piping!

One of the key feature of the **RxJS** library is the ability to **modify, alter, filter, and otherwise transform streams** into other streams.

This is similar to the `|` (pipe) operator in the command line, which allows you to pass the result of a command as input to another command:

```shell
$> `ls` -la . | `grep` txt
```
> Here, the result of the `ls` command is used as the input for the `grep` command

Coincidentally, every `Observable` has a method named `pipe(...)`, that can be used to apply a sequence of operations on this `Observable` and/or its emitted values.

An operation is nothing more than a function, called `Operator`, that takes an `Observable` as input, does something based on it, and outputs the results in a _new_ `Observable`.

> This output `Observable` will be used as input for the next `Operator` in the sequence, or be subscribed to if it's the last one.

### Plumbing 101

In our example code, we used a sequence of `Operator`s and applied it on the `Observable` produced by `fromEvent`, using the `pipe(...)` method:

```js
fromEvent(document.getElementById("myButton"), "click")
  // Call the pipe method on our initial Observable
* .pipe(
    // Apply the filter operator
    `filter`(/* ... */),
    // Then apply the map operator
    `map`(/* ... */)
* )
  // Subscribe to get the final result
  .subscribe(/* ... */);
```
> In this case, the `subscribe(...)` method is called on the `Observable` returned by the `map` operator, not the `fromEvent` function.

It's now time to see what the `filter` and `map` operators do...

### `filter` operator

For each value emitted by the source `Observable`, this operator checks wether it respects a user-defined condition. The `Observable` returned by `filter` will only emit the values that pass the check.

Here's a marble representation of this operator:

<p class="shadow center"><img src="marbles/filter_operator.svg" width="80%" /></p>

And how you would use it:

```js
randomLetters()
  .pipe(`filter(letter => letter === 'A')`)
  .subscribe(a => console.log(a));
```
> See [the official documentation][rxjs-filter]

#### In context

In our example, we want to filter `click` events based on wether the `Shift` key was pressed.

As a marble diagram, this would look like this:

> `C` stands for "Click (without `Shift` key)", `SC` for "`Shift` + Click"

<p class="shadow center"><img src="marbles/shift_click.svg" width="80%" /></p>

And in the code:

```js
fromEvent(document.getElementById("myButton"), "click")
  .pipe(`filter(click => click.shiftKey)`)
  .subscribe(click => console.log('Shift key pressed!', click))
```

### `map` operator

The `map` operator takes each emitted value of the source Observable, apply it a user-defined function, and emits the result of this function.

> This is similar to the `map(...)` method of an `Array`

Here's a marble representation of this operator:

<p class="shadow center"><img src="marbles/map_operator.svg" width="80%" /></p>

And how you would use it:

```js
randomLetters()
  .pipe(`map(letter => letter.toLowerCase())`)
  .subscribe(letter => console.log(letter));
```
> See [the official documentation][rxjs-map]

#### In context

In our example, for each shift-click event, we want a custom object containing only the mouse coordinates.

As a marble diagram, this would look like this:

<p class="shadow center"><img src="marbles/coordinates.svg" width="80%" /></p>

And in the code:

```js
shiftClick()
  .pipe(`map(click => ({ x: click.pageX, y: click.pageY }))`)
  .subscribe(coords => console.log('Received coordinates!', coords))
```
### All together now

In light of this new knowledge, let's take another look at the complete feature code:

```js
const { fromEvent } = rxjs;
// Operators are accessible through the rxjs.operators object.
*const { filter, map } = rxjs.operators;

fromEvent(document.getElementById("myButton"), "click")
  .pipe(
*   filter((click) => click.shiftKey),
*   map((event) => ({ x: event.pageX, y: event.pageY }))
  )
  .subscribe((coordinates) =>
    alert(\`Clicked at [${coordinates.x}, ${coordinates.y}]`)
  );
```

#### Marble-ous

As a marble diagram, this would look like this:

> User clicks on the button (with or without the `Shift` key)

<p class="shadow center"><img src="marbles/feature.svg" width="90%" /></p>

> Display coordinates in an `alert` using `Obj`

## What are the advantages

This `pipe` and `Operators` approach has several advantages:

1. The `subscribe` callback **only contains the code for the actual operation** we want to achieve in the end.
1. The list of operators breaks down in a readable way **which operations are applied and in which order**.
1. You can store each `Operator`'s produced `Observable` in its own variable, to enhance reusability and/or readability:
  ```js
  const `clickObs` = fromEvent(document.getElementById("myButton"), "click");

  const `shiftClickObs` = `clickObs`.pipe(filter((click) => click.shiftKey));

  const `coordinatesObs` = `shiftClickObs`.pipe(
      map((click) => ({ x: click.pageX, y: click.pageY }))
  );

  `coordinatesObs`.subscribe(
      alert(\`Clicked at [${coordinates.x}, ${coordinates.y}]`)
  );
  ```
1. Since many `Operator`s takes a callback function as parameter, you can easily **refactor your code in small reusable functions**, that could live in their own module _(see next slide)_

### Function refactor

```js
const { fromEvent } = rxjs;
const { filter, map } = rxjs.operators;

fromEvent(document.getElementById("myButton"), "click")
  .pipe(
    filter(`clickWithShift`),
    map(`mouseCoordinates`)
  )
  .subscribe(`displayCoordinates`);

function `clickWithShift`(click) {
  return click.shiftKey;
}

function `mouseCoordinates`(event) {
  return {
    x: event.pageX,
    y: event.pageY
  };
}

function `displayCoordinates`(coordinates) {
  alert(\`Clicked at [${coordinates.x}, ${coordinates.y}]`);
}
```

## Why would I use this... thing?

We already discussed the topic of using **callbacks** to do so, and seen that it could rapidly [get out of hands][callback-hell] when multiple async operations are to be executed one after the other.

So we learnt about **[Promises][js-prom]**, wich aleviate the pains of callbacks and allow setting up more controlled flow of actions. But depending on the use case, Promises would not be adequate.

Observables (and reactive programming) are nothing more than **another tool to work with async operations**. They have some **similarities** with [Promises][js-prom], but here's the **key differences**:

| `Promises` | `Observables` |
| --- | --- |
| foo | bar |

## Resources

- [JavaScript Theory: Promise vs Observable][prom-vs-obs]

[from-event]: https://rxjs-dev.firebaseapp.com/api/index/function/fromEvent
[destructuring]: ../js/#38
[htmlshell]: http://htmlshell.com/
[callback-hell]: https://miro.medium.com/max/1400/0*bO_JSfydCKFUnJ2d.png
[js-prom]: ../js-promises
[prom-vs-obs]: https://medium.com/javascript-everyday/javascript-theory-promise-vs-observable-d3087bc1239a
[rx]: http://reactivex.io/
[rxjs-operators]: https://rxjs-dev.firebaseapp.com/api/operators
[rxjs-filter]: https://rxjs-dev.firebaseapp.com/api/operators/filter
[rxjs-map]: https://rxjs-dev.firebaseapp.com/api/operators/map