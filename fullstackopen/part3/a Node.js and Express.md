# Node.js and Express

Browsers don't yet support the newest features of JavaScript, and that is why the code running in the browser must be *transpiled*, eg. with babel. With JavaScript on the backend, though, the newest versions of Node support a large majority of the latest features of JavaScript, so we can use those features without having to transpile our code.

To create our backend Node projects, we will use npm. Navigate to an empty directory and use the `npm init` command, which will auto-generate a *package.json* file at the root of the project that contains information about the project.

We will want to add the `start` script to the *scripts* object in *package.json*:
```
{
  // ...
  "scripts": {
    "start": "node index.js",    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ...
}
```

## Simple web server

We can create a simple web server by adding this code to `index.js`:
```
const http = require('http')

const app = http.createServer((request, response) => {
  response.writeHead(200, { 'Content-Type': 'text/plain' })
  response.end('Hello World')
})

const PORT = 3001
app.listen(PORT)
console.log(`Server running on port ${PORT}`)
```

We can visit our application in the browser at http://localhost:3001.

In the first row, we import Node's built-in web server module. The `require` syntax is for a CommonJS module. While code in the browser uses ES6 modules, Node.js uses CommonJS modules, which were implemented in Node before JavaScript supported them. There are differences, but we will not explore them in this course.

The code uses `createServer` to create a new web server. An *event handler* is registered to the server that is called *every time* an HTTP request is made to the server's address. This event handler writes the response with headers with a status code 200 and the *Content-Type* header set to *text/plain*, and the contents to be *Hello World*.

The last rows bind the http server assigned to the `app` variable to listen to HTTP requests sent to port 3001.

# Express

Many libraries have been developed to ease server-side development with Node, by offering a more pleasing interface to work with the built-in http module. These libraries aim to provide a better abstraction for general use cases we usually require to build a backend server. By far, the most popular library intended for this purpose is express.

We can install express using the command `npm install express`.

In the dependencies in the *package.json* file, we can see the dependency for express:
```
"express": "^4.17.2"
```

The versioning model used in npm is called semantic versioning. The caret in the front of *^4.17.2* means that if and when the dependencies of a project are updated, the version of express that is installed will be at least *4.17.2*. However, the installed version of express can also have a larger *patch* number (the last number), or a larger *minor* number (the middle number). The major version of the library, indicated by the first *major* number, must be the same.

We can update the dependencies of the project with the command `npm update`.

If we start working on the project on another computer, we can install all up-to-date dependencies of the project by running the command `npm install`.

If the *major* number of a dependency does not change, then the newer versions should be backwards compatible. This means that if our application happened to use version 4.99.175 of express in the future, then all the code implemented in this part would still have to work without making changes in the code. In contrast, the future 5.0.0 version of express may contain changes that would cause our application to no longer work.

# Web and express

Here is an example notes server using Express:
```
const express = require('express')
const app = express()

let notes = [
  ...
]

app.get('/', (request, response) => {
  response.send('<h1>Hello World!</h1>')
})

app.get('/api/notes', (request, response) => {
  response.json(notes)
})

const PORT = 3001
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
``` 

First, we need to import express, which is a *function* used to create an express application, which we store in the `app` variable. We then define two *routes* to the application. 

The first defines an event handler used to handle HTTP GET requests made to the application's / root. The request parameter contains all of the information of the HTTP request, and the second response parameter is used to define how the request is responded to. In the above code, we use the send method of the response object, which we use to send a string containing HTML, so express automatically sets the value of the *Content-Type* header to be *text/html*. The status code defaults to 200.

The second route defines an event handler that handles HTTP GET requests made to the *notes* path of the application. The request is responded to with the json method of the response object. This will send the notes array passed to it as a JSON formatted string. Express automatically sets the *Content-Type* header with *application/json*.

With express, we don't need to transform the data with `JSON.stringify` like before, express handles the transformation automatically.

You can use the node-repl by typing `node` in the command line. REPL stands for Read-Eval-Print-Loop. You can use it for testing how commands work while you're writing application code.

## nodemon

In normal Node applications, changes will not appear unless you close the program and reopen it. You can fix this problem by using nodemon. nodemon will watch the files in the directory in which nodemon was started, and if any files change, nodemon will automatically restart your node application.

It should be installed as a *development dependency*: `npm install --save-dev nodemon`.

We can start our application using nodemon like this:
```
node_modules/.bin/nodemon index.js
```

Changes to the application code now cause the server to restart automatically. Though the backend server restarts automatically, the browser still has to be manually refreshed.

To specify a script in *package.json*, we can write `"dev": "nodemon index.js"` into the scripts object. We can forego the path to nodemon, because npm automatically knows to search for the file from that directory. It can now be run using the command `npm run dev`.

## REST

Representational State Transfer, aka REST, is an architectural style meant for building scalable web applications.

In RESTful thinking, singular things are called resources. Every resource has an associated URL which is the resource's unique address.

One convention for creating unique addresses is to combine the name of the resource type with the resource's unique identifier.

For example, if we have a root URL of *www.example.com/api*, our resource type of note to be *notes*, then the address of a note resource with the identifier 10, has the unique address *www.example.com/api/notes/10*. The URL for the entire collection of all note resources is *www.example.com/api/notes*.

We can execute different operations on resources, and the operation to be executed is defined by the HTTP *verb*:

| URL | verb | functionality |
| --- | --- | --- |
| notes/10 | GET | fetches a single resource |
| notes | GET | fetches all resources in the collection |
| notes | POST | creates a new resource based on the request data |
| notes/10 | DELETE | removes the identified resource |
| notes/10 | PUT | replaces the entire identified resource with the request data |
| notes/10 | PATCH | replaces a part of the identified resource with the request data |

This is how we manage to roughly define what REST refers to as a uniform interface, which means a consistent way of defining interfaces that makes it possible for systems to cooperate.

This way of interpreting REST falls under the second level of RESTful maturity in the Richardson Maturity Model. According to the original definition, we have not defined a REST API. In fact, a large majority of "REST" APIs do not meet the original criteria either. In some places, this model will be called a CRUD API, an example of resource-oriented architecture.

## Fetching a single resource

To create a route for fetching a single resource, we will need to use an id parameter to create a unique address for each resource. We can define parameters for routes in express by using the colon syntax. Here is an example route:
```
app.get('/api/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  const note = notes.find(note => note.id === id)
  
  if (note) {    
    response.json(note)  
  } else {    
    response.status(404).end()  
  }
})
```

This route will handle all HTTP GET requests that are of the form */api/notes/SOMETHING*, where *SOMETHING* is an arbitrary string.

The *id* parameter in the route of a request can be accessed through the request object:
```
request.params.id
```
Note that we must cast the id into a Number, because the params in the request are strings.

The if statement at the end handles the case when we search for an id that does not exist, in which case, `const note` will be undefined. If no note is found, the server should respond with the status code 404 Not Found instead of 200. Since no data is attached to the response, we use the *status* method for setting the status, and the *end* method for responding to the request without sending any data.

We do not need to display anything when there is a 404 in this case, because REST APIs are interfaces intended for programmatic use, so the error status code is all that is needed.

## Deleting resources

Here is an example route for deleting a resource using an HTTP DELETE request:
```
app.delete('/api/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  notes = notes.filter(note => note.id !== id)

  response.status(204).end()
})
```

If deleting the resource is successful, meaning the resource exists and is removed, we respond to the request with the status code 204 No Content and return no data with the response.

There is no consensus on what status code should be returned to a DELETE request if the resource does not exist. The only two options are 204 and 404. We will use 204 in both cases for simplicity.

## Postman

Postman is a tool that can be used to test your backend application.

## Visual Studio Code REST Client

The VS Code REST client plugin is an alternative to using Postman.

To use it, we create a new *requests* folder in the root of the application, and add files with the *.rest* extension to send requests. For example, to get all notes, the file would contain:
```
GET http://localhost:3001/api/notes
```

If you then click the *Send Request* text just above the code, the REST client will execute the HTTP request and the response from the server will be opened in the editor.

Here is an example of a POST request:
```
POST http://localhost:3001/api/notes HTTP/1.1
content-type: application/json

{
    "content": "Testing 123",
    "important": true
}
```

You can add multiple requests in the same file using ### separators.

## Receiving data

To add resources, we need to make an HTTP POST request to the address containing the list of all of that type of resource. We also need to send the information in the request body in JSON format.

For the backend to access that data easily, we will use the express json-parser, using the command `app.use(express.json())` to initialize it before using it.

For example:
```
const express = require('express')
const app = express()

app.use(express.json())
//...

const generateId = () => {
  const maxId = notes.length > 0
    ? Math.max(...notes.map(n => n.id))
    : 0
  return maxId + 1
}

app.post('/api/notes', (request, response) => {
  const body = request.body

  if (!body.content) {
    return response.status(400).json({ 
      error: 'content missing' 
    })
  }

  const note = {
    content: body.content,
    important: body.important || false,
    date: new Date(),
    id: generateId(),
  }

  notes = notes.concat(note)

  response.json(note)
})
```

The event handler function can access the data from the *body* property of the `request` object.

Without the json-parser, the *body* property would be undefined. The json-parser functions so that it takes the JSON data of a request, transforms it into a JavaScript object, and then attaches it to the *body* property of the `request` object before the route handler is called.

The post route above checks to make sure the *content* property exists, otherwise it will respond with the status code 400 Bad Request. If the *content* property has a value, the note will be based on the received data. Note that it generates a unique  ID for each new note, which is important for use in React. The date is also set by the server rather than the sending browser, since we can't trust that the browser has its clock set correctly.

