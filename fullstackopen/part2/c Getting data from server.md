# Getting data from server

Until now, we have only been working on the frontend, ie. client-side/browser functionality. We will begin working on backend, ie. server-side functionality soon.

We will start by using JSON Server to act as our server.

Use the following command to run the json-server:
```
npx json-server --port <port> --watch <JSON filename>
```

The port option specifies the port, otherwise 3000 will be default. The watch option automatically looks for any saved changes to the json file. `npx` is only needed if you did not install json-server globally.

If we use a file *db.json* with the contents:
```
{
    "notes": [
        {
        "id": 1,
        "content": "HTML is easy",
        "date": "2022-1-17T17:30:31.098Z",
        "important": true
        },
        {
        "id": 2,
        "content": "Browser can execute only JavaScript",
        "date": "2022-1-17T18:39:34.091Z",
        "important": false
        },
        {
        "id": 3,
        "content": "GET and POST are the most important methods of HTTP protocol",
        "date": "2022-1-17T19:20:14.298Z",
        "important": true
        }
    ]
}
```

We can access it by going to the address `http://localhost:<port>/notes`

Going forward, the React code will fetch data from the server and render it to the screen. When the data is edited, the React code also sends it to the server to make the change persist.

json-server stores all the data in the *db.json* file, which resides on the server. This is a useful way to simulate interacting with a database or server.

## The browser as a runtime environment

We first need to fetch the notes into our React application. We looked at `XMLHttpRequest` before, which was an HTTP request made using an XHR object. The use of XHR is no longer recommended.

JavaScript engines, or runtime environments, follow the asynchronous model. In principle, this requires all I/O operations (with some exceptions) to be executed as non-blocking. This means that code execution continues immediately after calling an I/O function without waiting for it to return.

When an asynchronous operation is completed, or more specifically, at some point after its completion, the JavaScript engine calls the event handlers registered to the operation.

Currently, JavaScript engines are *single-threaded*, which means that they cannot execute code in parallel. As a result, it is a requirement in practice to use a non-blocking model for executing I/O operations. Otherwise, the browser would freeze during, for instance, the fetching of data from a server.

Another consequence of the single-threaded nature of JavaScript engines is that if some code execution takes up a lot of time, the browser will get stuck for the duration of the execution. For the browser to remain responsive, the code logic needs to be such that no single computation can take too long.

In modern browsers, it is possible to run parallelized code with the help of *web workers*. The event loop of an individual browser window is still only handled by a single thread.

## npm

Two ways we could access the data from the server are using the promise based function `fetch`, or the `axios` library. We will use the `axios` library.

We can install it using npm: `npm install axios` or `npm i axios`

We can also install json-server as a dev-dependency: `npm i json-server --save-dev`

## Axios and promises
To use axios, you need to import it first: `import axios from 'axios'`

To get data from the server, we use the `get` method: 
`const promise = axios.get('http://localhost:3001/notes')`

Axios' method `get` returns a promise. A Promise is an object representing the eventual completion or failure of an asynchronous operation.

In other words, a promise is an object that represents an asynchronous operation. A promise can have three distinct states:
1. The promise is *pending*: It means that the final value (one of the following two) is not available yet.
2. The promise is *fulfilled*: It means that the operation has been completed and the final value is available, which generally is a successful operation. This state is sometimes also called *resolved*.
3. The promise is *rejected*: It means that an error prevented the final value from being determined, which generally represents a failed operation.

When we want to access the result of the operation represented by the promise, we must register an event handler to the promise. This is achieved using the method `then`:
```
const promise = axios.get('http://localhost:3001/notes')

promise.then(response => {
  console.log(response)
})
```

The JavaScript runtime environment calls the callback function registered by the `then` method providing it with a `response` object as a parameter. The `response` object contains all the essential data related to the response of an HTTP GET request, which would include the returned *data*, *status code*, and *headers*.

Storing the promise object in a variable is generally unnecessary, and it's instead common to chain the `then` method call to the axios method call, so that it follows it directly:
```
axios
  .get('http://localhost:3001/notes')
  .then(response => {
    const notes = response.data
    console.log(notes)
  })
```

The data returned by the server is plain text. The axios library is still able to parse the data into a JavaScript array, since the server has specified that the data format is *application/json; charset=utf-8* using the *content-type* header.

## Effect-hooks
The Effect Hook lets you perform side effects on function components. Data fetching, setting up a subscription, and manually changing the DOM in React components are all examples of side effects.

To use an effect, we import `useEffect` from React, and then need to provide *two parameters* to the function. The first is a function, the effect itself. By default, effects run after every completed render, but you can choose to fire it only when certain values have changed.

The second parameter of `useEffect` is used to specify how often the effect is run. If the second parameter is an empty array, then the effect is only run along with the first render of the component.

Here is an example of `useEffect` with an empty array, so it will only run on the first render of the component:
```
useEffect(() => {
  console.log('effect')
  axios
    .get('http://localhost:3001/notes')
    .then(response => {
      console.log('promise fulfilled')
      setNotes(response.data)
    })
}, [])
```