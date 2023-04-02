# Learn RxJS

[Learn RxJS - Documentation](https://www.learnrxjs.io/)

- [Learn RxJS](#learn-rxjs)
  - [Introduction](#introduction)
  - [Getting familiar with RxJS Primer](#getting-familiar-with-rxjs-primer)
    - [What is an Observable?](#what-is-an-observable)
    - [Subscription](#subscription)
    - [Operators](#operators)
    - [Pipe](#pipe)
    - [Operators can be grouped into common categories](#operators-can-be-grouped-into-common-categories)
      - [Creation operators](#creation-operators)
      - [Combination operators](#combination-operators)
      - [Error handling operators](#error-handling-operators)
      - [FIltering operators](#filtering-operators)

## Introduction

[RxJS](https://github.com/ReactiveX/rxjs) is one of the hottest libraries in web development today. Offering a powerful, functional approach for dealing with events and with integration points into a growing number of frameworks, libraries, and utilities, the case for learning Rx has never been more appealing. Couple this with the ability to utilize your knowledge across [nearly any language](http://reactivex.io/languages.html), having a solid grasp on reactive programming and what it can offer seems like a no-brainer.

**But...**

Learning RxJS and reactive programming is [hard](https://twitter.com/hoss/status/742643506536153088). There's the multitude of concepts, large API surface, and fundamental shift in mindset from an [imperative to declarative style](https://tylermcginnis.com/imperative-vs-declarative-programming/).

## Getting familiar with RxJS Primer

All the major concepts that I will need to gain a grasp on, and start being productive with RxJS.

### What is an Observable?

An Observable represents a stream, or source of data that can arrive over time. You can create an observable from nearly anything, but the most common use case in RxJS is from events. This can be anything from mouse moves, button clicks, input into a text field, or even route changes. The easiest way to create an observable is through the built in creation functions. For example, we can use the [fromEvent](https://www.learnrxjs.io/learn-rxjs/operators/creation/fromevent) helper function to create an observable of mouse click events:

```JS
//import the fromEvent operator
import { fromEvent } from 'rxjs';

//grab button reference
const button = document.getElementById('myButton');

//create an observable of button clicks
const myObservable = fromEvent(button, 'click');
```

At this point we have an observable but it's not doing anything. **This is because observables are cold, or do not activate a producer (like wiring up an event listener), until there is a...**

### Subscription

Subscriptions are what set everything in motion. You can think of this like a faucet, you have a stream of water ready to be tapped (observable), somone just needs to turn the handle. In the case of observables, that role belongs to the `subscriber`.

To create a subscription, you call the `subscribe` method, supplying a function (or object) - also known as an `observer`. this is where you can decide how to **react**(-ive programming) to each event. Let's walk through what happens in the previous scenario when a subscription is created:

```JS
//import { fromEvent } from 'rxjs';

//grab button reference
const button = document.getElementById('myButton');

//create an observable of button clicks
const myObservable = fromEvent(button, 'click');

//for now, let's just log the event on each click
const subscripton = myObservable.subscribe(event => console.log(event));
```

In the example above, callling `myObservable.subscribe()` will:

1. Set up an event listener on our button for click events.
2. Call the function we passed to the subscribe method (observer) on each click event.
3. Return a subscription object with an `unsubscribe` which contains clean up logic, like removing appropriate event listeners.

The subscribe method also accepts an object map to handle the case of error or completion. You probably won't use this as much as simply supplying a function, but it's good to be aware of should the need arise:

```JS
//instead of a function, we will pass an object with next, error, and complete methods
const subscription = myObservable.subscribe({
  //on successful emissions
  next: event => console.log(event);

  //on errors
  error: error => console.log(error);

  //called once on completion
  complete: () => console.log('complete!');
});
```

It's important to note that each subscription will create a new execution context. This means calling `subscribe` a second time will create a new event listener:

```JS
//addEventListener called
const subscription = myOservable.subscribe(event => console.log(event));

//addEventListner called again!
const secondSubscription = myObservable.subscribe(event => console.log(event));

//clean up with unsubscribe
subscription.unsubscribe();
secondSubscription.unsubscribe();
```

by default, a subscription creates a one on one, one-sided conversation between the observable and observer. This is like your boss (the observable) yelling (emitting) at you (the observer) for merging a bad PR. This is also known as **unicasting**. If you prefer a conference talk scenario - one observable, many observers - you will take a different apprach which includes **multicasting** with `Subjects` (either explicitly or behind the scenes).

It's worth noting that when an Observable emits data to observers this is a push based model. The source doesn't know or care what subscribers do with the data, it simply pushes it down the line.

While working on a stream of events is nice, it's only so useful on its own. **What makes RxJS the "lodash for events" are its...**

### Operators

Operators offer a way to manupulate values from a source, returning an observable of the transformed values. Many of the RxJS operators will look familiar if you are used to JavaScripts `Array` methods. For instance, if you want to transform emitted values from an observable source, you can use [map](https://www.learnrxjs.io/learn-rxjs/operators/transformation/map):

```JS
import { of } from 'rxjs';
import { map } from 'rxjs/operators';

/*
 * 'of' allows you to deliver values in a serquence
 * in this, case, it will emit 1,2,3,4,5 in order.
*/

const dataSource = of(1, 2, 3, 4, 5);

subscription = dataSource
  .pipe(
    // add 1 to each emitted value
    map(value => value + 1)
  )
  //log: 2,3,4,5,6
  .subscribe(value => console.log(value));
```

Or if you want to filter for specific values, you can use [filter](https://www.learnrxjs.io/learn-rxjs/operators/filtering/filter):

```JS
import { of } from 'rxjs';
import { filter } from 'rxjs/operators';

const dataSource = of(1, 2, 3, 4, 5);

subscription = dataSource
  .pipe(
    // only accpt values 2 or greater
    filter(value => value >= 2)
  )
  //log: 2,3,4,5
  .subscribe(value => console.log(value));
```

In practice, if there is a problem you need to solve, it's more than likely **there is an operator for that**. And while the sheer number of operators can be overwhelming as you being your RxJS journey, you can narrow it down to a small handful (and we will) to start being effective. Over time, you will come to appreciate the flexibility of the operator library when obscure scenarios inevitable arrive.

**One thing you may have noticed in the example above, is operators exist within a...**

### Pipe

The `pipe` function is the assembly line from your observable data source through your operators. Just like raw material in a factory goes through a series of stops before it becomes a finished product, source data can pass through a `pipe`-line of operators where you can manipulate, filter, and transform the data to fit your use case. it's not uncommon to use 5 (or more) operators within an observable chain, contained within the `pipe` function.

For instance, a type-ahead solution built with observables may use a group of operators to optimize both the request and display process:

```JS
//observable of values from a text box, pipe chains operators together
inputValue
 .pipe(
  //wait for a 200ms pause
  debounceTime(200),
  //if the value is the same, ignore
  distinctUntilChanged(),
  //if an updated value comes through while request is still active cancel previous request and 'switch' to new observable
  switchMap(searchTerm => typeaheadApi.search(searchTerm))
 )
 //create a subscription
  .subscribe( results => {
    //update the dom
  });
```

**But how do you know which operator fits your use-case? the good news is...**

### Operators can be grouped into common categories

The first stop when looking for the correct operator is finding a related category. Need to filter data from source? Check out the [filtering](https://www.learnrxjs.io/learn-rxjs/operators/filtering) operators. Trying to track down a bug, or debug the flow of data through your observable stream? there are [utility](https://www.learnrxjs.io/learn-rxjs/operators/utility) operators that will do the trick. **The operator categories include...**

#### [Creation operators](https://www.learnrxjs.io/learn-rxjs/operators/creation)

These operators allow the creation of an observable from nearly anything. From generic to specific use-cases you are free to turn everything into a stream.

For example, suppose we are creating a progress bar as user scrolls through an article. We could turn the scroll event into a stream by utilizing the [fromEvent](https://www.learnrxjs.io/learn-rxjs/operators/creation/fromevent) operator:

```JS
fromEvent(scrollContainerElement, 'scroll'){
  .pipe(
    //we will discuss cleanup strategies like this in future article
    takenUntil(userLeavesArticle)
  )
  .subscribe(event => {
    // calculate and update DOM
  });
}
```

The most commonly used creation operators are [of](https://www.learnrxjs.io/learn-rxjs/operators/creation/of), [from](https://www.learnrxjs.io/learn-rxjs/operators/creation/from), and [fromEvent](https://www.learnrxjs.io/learn-rxjs/operators/creation/fromevent).

#### [Combination operators](https://www.learnrxjs.io/learn-rxjs/operators/combination)

The combination operators allow the joining of information from multiple observables. Order, time, and structure of emitted values is the primary variation among these operators.

For example, we can combine updates from multiple data sources to perform a calculation:

```JS
//give me the last emitted value from each source, whenever either source emits
combineLatest(sourceOne, sourceTwo).subscribe(
  ([latestValueFromSourceOne, latestValueFromSourceTwo]) => {
    // perform calculation
  }
);
```

The most commonly used combination operators are [combineLatest](https://www.learnrxjs.io/learn-rxjs/operators/combination/combinelatest), [concat](https://www.learnrxjs.io/learn-rxjs/operators/combination/concat), [merge](https://www.learnrxjs.io/learn-rxjs/operators/combination/merge), [startWith](https://www.learnrxjs.io/learn-rxjs/operators/combination/startwith), and [withLatestForm](https://www.learnrxjs.io/learn-rxjs/operators/combination/withlatestfrom).

#### [Error handling operators](https://www.learnrxjs.io/learn-rxjs/operators/error_handling)

The error handling operators provide effective ways to gracefully handle errors and perform retries, should they occur.

For example, we can use [catchError](https://www.learnrxjs.io/learn-rxjs/operators/error_handling/catch) to safeguard against failed network requests:

```JS
source
 .pipe(
    mergeMap(value =>{
      return makeRequest(value).pipe(
        catchError(handleErrorByReturningObservable)
      );
    })
 )
 .subscribe(value => {
  //take action
 });
```

The most commonly used error handling operators is [catchError](https://www.learnrxjs.io/learn-rxjs/operators/error_handling/catch).

#### [FIltering operators](https://www.learnrxjs.io/learn-rxjs/operators/filtering)

The filtering operators provide techniques for accepting - or declining - values from an observable source and dealing with backpressure, or the build up of values within a stream.

For example, we can use the [take](https://www.learnrxjs.io/learn-rxjs/operators/filtering/take) operator to capture only the first 5 emitted values from a source:

```JS
source.pipe(take(5)).subscribe(value => {
  // take action
});
```

The most commonly used filtering operators are [debounceTime](https://www.learnrxjs.io/learn-rxjs/operators/filtering/debouncetime), [distinctUntilChanged](https://www.learnrxjs.io/learn-rxjs/operators/filtering/distinctuntilchanged), [filter](https://www.learnrxjs.io/learn-rxjs/operators/filtering/filter), [take](https://www.learnrxjs.io/learn-rxjs/operators/filtering/take), and [takeUntil](https://www.learnrxjs.io/learn-rxjs/operators/filtering/takeuntil)
