# Get started transforming streams with map, pluck, and mapTo

- [Get Started](#get-started-transforming-streams-with-map-pluck-and-mapto)
  - [Introducing map](#introducing-map)

When working with observables one of the most common use cases you will encounter is the need to transform a stream of some value type into a stream of another value type.
For instance, you may have an observable of click events that you wish to transform into an observable of objects containing just the `clientX` and `clientY` coordinates.
Or maybe you need to extract a value from a stream of input events to perform a calculation or initiate a request. Or perhaps you just want to extract a single property
from an object, like a key code, to perform another action down the (pipe)line. The scenarios for transforming streams are endless.

Here we will learn about the most common operator to transform streams, the `map` operator. We'll start with taking a look at `Array.map` to build a general understanding of the `map` operation.
Next, we will explore how we can apply this approach to observables by using the RxJS `map` operator. Finally, we will check out a few helper operators that can be used in place
of `map` should the right scenario present itself, exploring common use cases along the way.

## Introducing `map`

When dealing with arrays, the `map` method lets you transform an array by applying a provided function (often referred to as a 'projection' function) to each item within the array. For instance, let's say we have an array of numbers `1-5`:

```JS
const numbers = [1, 2, 3, 4, 5];
```

If we wanted to transform this into an array of each number multiplied by ten, we could use the `map` method. To do this, we call `map` on our numbers array, passing it a function which will be invoked with each value of the source array, returning the number multiplied by ten:

```JS
const numbers = [1, 2, 3, 4, 5];
const numbersTimesTen = numbers.map(number => number * 10);

// [10,20,30,40,50]
console.log(numbersTimesTen)
```

The `map` method does not mutate the existing array, but instead returns a new array. For example, if we were to log the `numbers` array after calling `map`, we can see that it's unchanged:

```JS
const numbers = [1, 2, 3, 4, 5];
const numbersTimesTen = numbers.map(number => number * 10);

// [10,20,30,40,50]
console.log(numbersTimesTen)

// [1,2,3,4,5]
console.log(numbers);
```

To understand this better, let's walk through what a naive implementation of `Array.map` could look like.

1. We create a new array.
2. For every item contained in the source array we apply the provided function.
3. We then push the result of this function to a temporary `resultArray`.
4. After doing this for every item, we return the new array.

```JS
Array.prototype.map = function(projectFn) {
 let resultArray = [];

 // loop through each item
 this.forEach(item => {
  // apply the provided project function
  let result = projectFn(item);
  // push the result to our new array
  resultArray.pish(result);
 });
 // return the array containing transformed values
 return resultArray;
}
```
