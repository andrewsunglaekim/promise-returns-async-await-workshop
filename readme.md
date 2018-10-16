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

## Discussion 1 - A simple promise

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

p.then((response) => {
  console.log('then response of p: ', response);
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

> For the purposes of this workshop, we'll just be using simple `get` requests with axios to highlight return values within promises.

## Returning a get response from axios

Before getting to heavy into the JS, lets just make a quick api request to the mapquest developer api. Open your browser and enter the following url:

```
http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures
```

> replace <api-key-here> with your api key

We'll see a pretty generic api response with mapquests data on a location query of 'redventures'. Let's see how we can leverage this response inside JS promises.

To get started, let's use this skeleton to create a simple page that imports the axios dependency

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Promises</title>
    <meta charset="us-ascii">
  </head>

  <body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.18.0/axios.min.js"></script>
    <script>

    </script>
  </body>
</html>
```

> The following code snippets should just be placed inside the empty script tag after the axios cdn import.

Now let's make a simple get request to the mapquest developer api:

```js
const promise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures');
```

Unfortunately this line in and of itself doesn't do too much for us. It does make the request, however, we need to add handlers to it to actually perform some action based off of the results.

We could do something like this if we just want to see the response or error in our logs:

```js
const promise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures')
  .then((res) => {
    console.log(res);
  })
  .catch((err) => {
    console.log(err);
  });
```

## Discussion 2 - return values in promises

Look at the following code and discuss the following in your group. Assuming the request is successful, what will be logged in the second `then`:

```js
const promise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures')
  .then((res) => {
    console.log(res);
  })
  .catch((err) => {
    console.log(err);
  });

promise.then((res) => {
  console.log(res);
})
```


In the first `then` call back we are logging out the response. If we want to encapsulate the promise's response again we need to make sure we return the value in the original `then`.

### example

Let's say for contrived purposes, we wanted to log the response two different times within two separate `then`'s

```js
const promise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures')
  .then((res) => {
    console.log(res);
    return res;
  })
  .catch((err) => {
    console.log(err);
  });

promise.then((res) => {
  console.log(res);
});
```

> We should note here that anything could have been returned the first `then`. Often times an initial then is used to aggregate or make objects based off part of the response. Most importantly though, subsequent `then`'s will always leverage whatever value was returned in the `then` before it.

### Exercise 1

## Discussion 3 - Promise.all()

Often times when developing, we will need to make several api calls. Sometimes, we need to make these calls all at once but want to do something once everything has been returned. Enter, `Promise.all()`.

Look at the following code snippet with your groups and discuss how we think it will execute:

```js
const rvPromise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures');
const charlottePromise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=charlotte,nc');

Promise.all([rvPromise, charlottePromise]).then((responses) => {
  console.log(responses);
})
```

### Nesting promises

With `Promise.all` we know all the information up front to make all the calls. We just want to make sure our `then` happens after they all return.

However, sometimes we need to be able to make a call to retrieve some data in order to make a subsequent call.

An example we use in Novant: We query a search to bring back health care providers. Based on the providers we get back, we make an additional call to retrieve schedules for those providers.

### A contrived example

For contrived purposes were going to use the geocoding api endpoint to provide us with a lat/long combo then pass that into a subsequent reverse geocoding endpoint to gives us ... the same response.

```js
axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures')
  .then((res) => {
    console.log(res);
    const latLng = res.data.results.locations[0].latLng;
    axios.get(`http://www.mapquestapi.com/geocoding/v1/reverse?key=<api-key-here>&location=${latLng.lat},${latLng.lng}`)
      .then((reverseRes) => {
        console.log(reverseRes);
      })
  });
```
