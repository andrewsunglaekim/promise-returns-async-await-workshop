# Promise returns and Async/await

> The learning objectives are links to anchors within this lesson plan that try to execute the learning objective. Additionally reference links are peppered throughout the headers and content of this lesson plan that contain information pertinent to the where the link is being used.

## Learning Objectives
- [Define a promise](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#discussion-1---a-simple-promise-35)
- [Leverage `.then` to handle successful async functionality](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#discussion-1---a-simple-promise-35)
- [Leverage `.catch` to handle unsuccessful async functionality](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#discussion-1---a-simple-promise-35)
- [Leverage `.finally` to handle any async functionality](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#discussion-1---a-simple-promise-35)
- [Pass returned values to chained promises](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#discussion-2---return-values-in-promises-530)
- [Use `Promise.all` to handle multiple async functionalities](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#discussion-3---promiseall-550)
- [Use promise chaining to nest async functionality](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#nesting-promises-1060)
- [Identify when you would choose `Promise.all` or promise chaining](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#nesting-promises-1060)
- [Leverage async/await as syntactic sugar for promises to clean up promise chains](https://github.com/andrewsunglaekim/promise-returns-async-await-workshop#asyncawait-1080)


## Framing (2/2)

As developers, we implement/debug lots of asynchronous code. Often times the challenges we face has more to do with data returns then the asynchronicity it self. This workshop is designed to help understand return values in async programming.

## [Discussion 1 - A simple promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) (3/5)

As per the MDN docs, [a promise is a returned object to which you attach callbacks](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

Examine the following code for 1 minute:

```js
const p = new Promise((resolve, reject) => {
  const randomRoll = Math.random();
  if (randomRoll > 0.5) {
    resolve('This is the resolved value');
  } else {
    reject('This is the rejected value');
  }  
}).then((response) => {
  console.log('Then response: ', response);
}).catch((error) => {
  console.log('Catch response: ', error);
}).finally((response) => {
  console.log('Finally response: ', response);
});
```

Spend 2 minutes discussing the following:
1. What could potentially be logged?
2. How does the `resolve` and `reject` functions influence the `.then` and `.catch` methods?

<details>
  <summary>Some Thoughts</summary>
  Everything is basically what we expect it to be. Whatever is passed to `resolve` and `reject` is handled respectively in `then` and `catch`. `finally`, however, will always result in an undefined parameter because you want this to happen either way regardless of success or failure.
</details>

## [Executor function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise#Syntax)(5/10)

The executor function of a new promise is executed immediately by the Promise implementation. The executor is called before the Promise constructor even returns the created object.

### Resolve
In the above example if the randomness rolls high, we `resolve('This is the resolved value');` Because of this we have access to the value passed into the `resolve` function in the `.then` function of the promise. Which is why we see the resolved value when it's logged in the callback of the `.then` function.

### Reject
Similarly, we see the value passed in to `reject` within the `.catch` callback's `console.log`.

More often then not, we don't really write many executor functions in our actual code base. We usually will leverage some library that extracts away the executor function portion and returns an instance of a promise object.

> Lookin at the above code,  with the use of the `new` keyword, we can clearly see that `Promise` is a constructor function. Which means `const p` is a new instance of a `Promise` object. Libraries we use just kind of do this out of the box

### Promise objects and schroedinger's callback (5/15)
[Schrödinger's cat](https://en.wikipedia.org/wiki/Schr%C3%B6dinger%27s_cat). It is a thought experiment to understand a paradox of not being able to qualify something because of the act of qualifying it. Similarly, we don't really know what the promise object is going to return until the asynchronous operation has finished. It can be both successful and unsuccessful until the response comes back.

The promise object represents the eventual completion(or failure) of an asynchronous operation, and its resulting value. In our case, our "eventual completions(or failure)" is happening synchronously through our executor(a simple `Math.random() > 0.5`), but in most cases the `resolve` and `reject` functions are called after asynchronous behavior executes. The executor function in all cases of a new promise object prescribes what causes a resolution or rejection through `resolve` and `reject`. For an example in the wild checkout axios source code [here](https://github.com/axios/axios/blob/master/dist/axios.js#L980)

We as developers are never sure how long our api fetches can take. We just send them out and give directions to our applications to execute later. If we don't know how long it will take we definitely don't know what the result is until it happens. Thats why we prescribe success(`then`) and failure(`catch`) handlers



## [axios](https://github.com/axios/axios)
Axios has kind of become the defacto standard for HTTP requests in the browser and node. We'll be using axios to do simple get requests. The libraries we use in this workshop are kind of irrelevant, the important thing to note is  how promises are leveraged to handle the asynchronous behavior. Axios abstracts the executor functions away from us, and allows us to interact with it methods by making them return promises.

> For the purposes of this workshop, we'll just be using simple `get` requests with axios to highlight return values within promises.

## Returning a get response from axios (10/25)

Before getting to heavy into the JS, lets just make a quick api request to the [mapquest geocoding api](https://developer.mapquest.com/documentation/geocoding-api/address/get/). Open your browser and enter the following url:

```
http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures
```

You can get your own mapquest api key [here](https://developer.mapquest.com/plan_purchase/steps/business_edition/business_edition_free/register)

> replace `<api-key-here>` with your api key. In the following code snippets all keys will be represented like it is in the above snippet. Remember to move the greater than and less than signs as well.

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

## [Discussion 2 - return values in promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then#Return_value) (5/30)

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
});
```

<details>
  <summary>Answer</summary>
  It logs undefined. In the first `then` call back we are logging out the response. If we want to encapsulate the promise's response again we need to make sure we return the value in the original `then`.
</details>


### example (5/35)

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

> We should note here that anything could have been returned the first `then`. Often times an initial then is used to aggregate or make objects based off part of the response. Most importantly though, subsequent `then`s' parameters will always leverage whatever value was returned in the `then` before it.

### [Exercise 1](https://github.com/andrewsunglaekim/promise-returns-async-await-ex) (10/45)

## [Discussion 3 - Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) (5/50)

Often times when developing, we will need to make several api calls. Sometimes, we need to make these calls all at once but want to do something once everything has been returned. Enter, `Promise.all()`.

Look at the following code snippet with your groups and discuss how we think it will execute:

```js
const rvPromise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures');
const charlottePromise = axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=charlotte,nc');

Promise.all([rvPromise, charlottePromise]).then((responses) => {
  console.log(responses);
});
```

<details>
  <summary>Answer</summary>
  Sometimes code better expresses functionality then raw definitions. This bit of code works exactly as we would expect. Promises are created one after the other but the callback does not occur until all return.
</details>


### [Nesting promises](https://javascript.info/promise-chaining) (10/60)

With `Promise.all` we generally know all the information up front to make all the calls. We just want to make sure our `then` happens after they all return.

However, sometimes we need to be able to make a call to retrieve some data in order to make a subsequent call.

An example we use in Novant: We query a search to bring back health care providers. Based on the providers we get back, we make an additional call to retrieve schedules for those providers.

### A contrived example

For contrived purposes were going to use the [geocoding api endpoint](https://developer.mapquest.com/documentation/geocoding-api/address/get/) to provide us with a lat/long combo then pass that into a subsequent [reverse geocoding endpoint](https://developer.mapquest.com/documentation/geocoding-api/reverse/get/) to gives us ... the same response.

```js
axios.get('http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=redventures')
  .then((res) => {
    console.log(res);
    const latLng = res.data.results[0].locations[0].latLng;
    axios.get(`http://www.mapquestapi.com/geocoding/v1/reverse?key=<api-key-here>&location=${latLng.lat},${latLng.lng}`)
      .then((reverseRes) => {
        console.log(reverseRes);
      })
      .catch((err) =>{
        console.log(err);
      });
  })
  .catch(err) => {
    console.log(err);
  };
```

An alternate and probably better way to write the above:

```js
getLatLng('redventures').then((latLng) => {
  reverseGeoFetch(latLng).then((res) => {
    console.log(res);
  });
});

function getLatLng(location){
  const latLngPromise = axios.get(`http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=${location}`)
    .then((res) => {
      console.log(res);
      const latLng = res.data.results[0].locations[0].latLng;
      return latLng;
    })
    .catch((err) => {
      console.log(err);
    });
  return latLngPromise;  
}

function reverseGeoFetch(latLng) {
  return axios.get(`http://www.mapquestapi.com/geocoding/v1/reverse?key=<api-key-here>&location=${latLng.lat},${latLng.lng}`)
    .then(res => res)
    .catch((err) =>{
      console.log(err);
    })
}
```

### [Exercise 2](https://github.com/andrewsunglaekim/promise-returns-async-await-ex) (10/70)

## `async/await` (10/80)

We are able to build relatively readable code when nesting promises. However, with the introduction of es6 async/await syntax, we can refactor the above even further. Sweet sweet syntactic sugar.

There are probably several ways to write the above code snippets using `async`/`await`. Here's one way:

```js
const latLng = await getLatLng('redventures');
const res = await reverseGeoFetch(latLng);
console.log(res);

async function getLatLng(location){
  try {
    const res = await axios.get(`http://www.mapquestapi.com/geocoding/v1/address?key=<api-key-here>&location=${location}`);
    const latLng = res.data.results[0].locations[0].latLng;
    return latLng;
  } catch (err) {
    console.log(err);
  }
}

async function reverseGeoFetch(latLng) {
  try {
    const res = await axios.get(`http://www.mapquestapi.com/geocoding/v1/reverse?key=<api-key-here>&location=${latLng.lat},${latLng.lng}`);
    return res;
  } catch (err) {
    console.log(err);
  }
}
```

The `async` keyword is used to denote functions that will contain asynchronous functionality. In order for something to be `await`'ed. It must be contained within an `async` functions' scope.

In the execution of an `async` function the `await` allows lines after it to only execute until after the functionality to the right of the `await` is finished. Essentially forcing promises and async behavior to appear as synchronous code.

The reality is that it doesn't condense too many lines of code. The increase in readability, however, is great.

### [Exercise 3](https://github.com/andrewsunglaekim/promise-returns-async-await-ex) (10/90)
