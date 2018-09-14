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
  if (randomRoll > 0.5) {
    resolve('This is the resolved value');
  } else {
    reject('This is the rejected value');
  }  
}).then((response) => {
  console.log(response);
}).catch((error) => {
  console.log(error);
});
```

Discuss the following:
1. What will be logged?
2. How does the resolve and reject functions influence the then and catch methods?

## axios
Axios has kind of become the defacto standard for HTTP requests in the browser and node. We'll be using axios to do simple get requests. The libraries we use in this workshop are kind of irrelevant, the important thing to note is  how promises are leveraged to handle the asynchronous behavior.
