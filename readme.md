# Promise returns and Async/await

## Learning Objectives
- Define a promise.
- Leverage `.then` to handle successful async functionality
- Leverage `.catch` to handle unsuccessful async functionality
- Pass returned values to chained promises
- Use `Promise.all` to handle multiple async functionality
- Use promise chaining to nest async functionality
- Identify when you would choose `Promise.all` or promise chaining
- Leverage async/await as syntactic sugar for promises to clean up promise chains.


[Essentially, a promise is a returned object to which you attach callbacks, instead of passing callbacks into a function.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

## Framing

As developers, we implement/debug lots of asynchronous code. Often times the challenges we face has more to do with data returns then the asynchronicity it self. This workshop is designed to help understand return values in async programming.

## Discussion - A simple promise

Examine the following code for 1 minute:

```js
const p = new Promise((resolve, reject) => {
  const randomRoll = Math.random()
  console.log(randomRoll);
  throw new Error('boop');
  if (randomRoll > 0.9) {
    resolve('This is the resolved value');
  } else {
    reject('This is the rejected value');
  }  
}).then((response) => {
  console.log('Then response: ', response);
}).catch((error) => {
  console.log('Catch response: ', error);
  throw new Error('boop');
}).finally((response) => {
  console.log('Finally response: ', response);
});

p.then((res) => {
  console.log('then res of p: ', res);
}).catch((err) => {
  console.log(('catch err of p: ', err));
})
```

Spend 2 minutes discussing the following:
1. What will be logged?
2. How does the `resolve` and `reject` functions influence the `.then` and `.catch` methods?

## [Executor function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise#Syntax)

The executor function of a new promise is executed immediately by the Promise implementation. The executor is called before the Promise constructor even returns the created object.

More often then not, we don't really write many executor functions in our actual code base. We usually will leverage some library that extracts away the executor function portion and returns an instance of a promise object.

> Lookin at the above code,  with the use of the `new` keyword, we can clearly see that `Promise` is a constructor function. Which means `const p` is a new instance of a `Promise` object. Libraries we use just kind of do this out of the box

### Promise objects and schroedinger's callback
The promise object represents the eventual completion(or failure) of an asynchronous operation, and its resulting value. In our case, our "eventual completions(or failure)" is happening synchronously through our executor(a simple `Math.random() > 0.5`), but in most cases the `resolve` and `reject` functions are called after asynchronous behavior executes. The executor function in all cases of a new promise object prescribes what causes a resolution or rejection through `resolve` and `reject`.

### Resolve
In the above example if the randomness rolls high, we `resolve('This is the resolved value');` Because of this we have access to the value passed into the `resolve` function in the `.then` function of the promise. Which is why we see the resolved value when it's logged in the callback of the `.then` function.

### Reject
Similarly, we see the value passed in to `reject` within the `.catch` callback's `console.log`.

## axios
Axios has kind of become the defacto standard for HTTP requests in the browser and node. We'll be using axios to do simple get requests. The libraries we use in this workshop are kind of irrelevant, the important thing to note is  how promises are leveraged to handle the asynchronous behavior. Axios abstracts the executor functions away from us, and allows us to interact with it methods by making them return promises.
