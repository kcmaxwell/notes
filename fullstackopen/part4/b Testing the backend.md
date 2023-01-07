# Testing the backend

In some situations, it can be beneficial to implement some of the backend tests by mocking the database instead of using a real database. One library that could be used for this is mongodb-memory-server.

Since our application's backend is still relatively simple, we will test the entire application through its REST API, so that the database is also included. This kind of testing where multiple components of the system are being tested as a group is called integration testing.

## Test environment

The convention in Node is to define the execution mode of the application with the *NODE_ENV* environment variable. In our current application, we only load the environment variables defined in the *.env* file if the application is *not* in production mode. It is common practice to define separate modes for development and testing.

Let's change the scripts in *package.json* so that when tests are run, *NODE_ENV* gets the value *test*:
```
"start": "NODE_ENV=production node index.js",
"dev": "NODE_ENV=development nodemon index.js",
...
"test": "NODE_ENV=test jest --verbose --runInBand"
```

We also added the runInBand option to the script. This option will prevent Jest from running tests in parallel. This is important when our tests are using our database.

We specified the mode of the application to be *development* in the dev script, and *production* in start.

There is a slight issue in the way we specified the mode of the application in our scripts: it will not work on Windows. We can correct this by installing the cross-env package as a development dependency:
```
npm i --save-dev cross-env
```

We can then achieve cross-platform compatibility by using cross-env in our npm scripts:
```
"scripts": {
    "start": "cross-env NODE_ENV=production node index.js",
    "dev": "cross-env NODE_ENV=development nodemon index.js",
    // ...
    "test": "cross-env NODE_ENV=test jest --verbose --runInBand",
},
```

If you are deploying this application, you might need to set cross-env to a production dependency:
```
npm i cross-env -P
```

Now we can modify the way that our application runs in different modes. For example, we could define the application to use a separate test database when running tests.

In most situations where multiple people are developing, using our MongoDB Atlas database would not be wise. It would be better to use Mongo in-memory or use a Docker container. For now, we will use MongoDB Atlas.

This is how we can set a different database based on the Node environment:
```
const MONGODB_URI = process.env.NODE_ENV === 'test'   
    ? process.env.TEST_MONGODB_URI  
    : process.env.MONGODB_URI
```

We only need to define a TEST_MONGODB_URI environment variable in our *.env* file.

## supertest

We'll use the supertest package to help write tests for testing the API. Install it as a development dependency:
```
npm i --save-dev supertest
```

Let's write our first test:
```
const mongoose = require('mongoose')
const supertest = require('supertest')
const app = require('../app')

const api = supertest(app)

test('notes are returned as json', async () => {
  await api
    .get('/api/notes')
    .expect(200)
    .expect('Content-Type', /application\/json/)
})

afterAll(() => {
  mongoose.connection.close()
})
```

The test imports the Express application from the *app.js* module and wraps it with the *supertest* function into a so-called superagent object. This object is assigned to the *api* variable, and tests can use it for making HTTP requests to the backend.

Our test makes an HTTP GET request to the *api/notes* url and verifies that the request is responded to with the status code 200. The test also verifies that the *Content-Type* header is set to *application/json*, indicating that the data is in the desired format.

Once all tests have finished running, we have to close the database connection used by Mongoose. We use the afterAll method to do this.

One console warning you might run into is: "Jest did not exit one second after the test run has completed." The problem is probably caused by Mongoose version 6.x, and Mongoose documentation does not recommend testing Mongoose applications with Jest. One way to get rid of this is to run tests with the extra option *--forceExit*.

Another error you may come across is your test takes longer than the default Jest test timeout of 5000 ms. This can be solved by adding a third parameter to the test function, the time in ms for the new timeout.

One important detail is that we extracted the Express application into *app.js*, and *index.js* launches the application at the specified port with Node's built-in *http* object. The tests only use the express application in *app.js*. This is because with supertest, if the server is not already listening for connections, then it is bound to an ephemeral port for you, so there is no need to keep track of ports. In other words, supertest takes care that the application being tested is started at the port it uses internally.

The middleware that outputs information about HTTP requests can obstruct the test execution output, so you can add a check so it does not print to the console in test mode.

## Initializing the database before tests

We can use the beforeEach function to initialize the database before each test:
```
const initialNotes = [  
    {
        content: 'HTML is easy',    
        date: new Date(),   
        important: false,  
    },  
    { 
        content: 'Browser can execute only Javascript',    
        date: new Date(),    
        important: true,  
    },
]

beforeEach(async () => {  
    await Note.deleteMany({})  
    let noteObject = new Note(initialNotes[0])  
    await noteObject.save()  
    noteObject = new Note(initialNotes[1])  
    await noteObject.save()
})
```

The database is cleared out at the beginning, and after that, we save the notes stored in *initialNotes* to the database. This way, the database is in the same state before every test is run.

## Running tests one by one

Our `npm test` script executes all of the tests for the application. When we are writing tests, it is usually wise to only execute one or two tests.

One good option is to specify the tests that need to be run as parameters of the *npm test* command. The following example only executes tests in the *tests/note_api.test.js* file:
```
npm test -- tests/note_api.test.js
```

The *-t option can be used for running tests with a specific name:
```
npm test -- -t "a specific note is within the returned notes"
```

The provided parameter can refer to the name of the test or the describe block. The parameter can also contain just a part of the name. The following command will run all of the tests that contain *notes* in their name:
```
npm test -- -t 'notes'
```

## async/await

The async/await syntax was introduced in ES7, and makes it possible to use *asynchronous functions that return a promise* in a way that makes the code look synchronous.

As an example, the fetching of notes from the database with promises looks like this:
```
Note.find({}).then(notes => {
  console.log('operation returned the following notes', notes)
})
```

The `Note.find` method returns a promise and we can access the result of the operation by registering a callback function with the `then` method.

All of the code we want to execute once the operation finishes is written in the callback function. If we wanted to make several asynchronous function calls in sequence, the situation would soon become painful. The asynchronous calls would have to be made in the callback. This would likely lead to complicated code.

By chaining promises, we could keep the situation partially under control, and create a fairly clean chain of `then` method calls. Here is an example of a then-chain:
```
Note.find({})
  .then(notes => {
    return notes[0].remove()
  })
  .then(response => {
    console.log('the first note is removed')
    // more code here
  })
```

The then-chain is alright, but we can do better. One way was to use generator functions, which were introduced in ES6. The syntax is a bit clunky and not widely used.

The `async` and `await` keywords introduced in ES7 bring the same functionality as the generators, but in an understandable and syntactically cleaner way.

We could fetch all of the notes in the database by utilizing the await operator like this:
```
const notes = await Note.find({})

console.log('operation returned the following notes', notes)
```

The code looks exactly like synchronous code. The execution of code pauses at `const notes = await Note.find({})` and waits until the related promise is *fulfilled*, and then continues its execution to the next line. When the execution continues, the result of the operation that returned a promise is assigned to the `notes` variable.

The then-chain delete remove above could be implemented using await like this:
```
const notes = await Note.find({})
const response = await notes[0].remove()

console.log('the first note is removed')
```

There are a few important details to pay attention to when using async/await syntax. To use the await operator with asynchronous operations, they have to return a promise. This is not a problem, as regular asynchronous functions using callbacks are easy to wrap around promises.

The await keyword can't be used anywhere. Using await is possible only inside of an async function. This means the above examples need to be placed inside async functions.

## Error handling and async/await

If there's an exception while handling our async function, we end up with an unhandled promise rejection, and the request never receives a response.

With async/await, the recommended way of dealing with exceptions is to surround it with a `try/catch` mechanism:
```
notesRouter.post('/', async (request, response, next) => {
  const body = request.body

  const note = new Note({
    content: body.content,
    important: body.important || false,
    date: new Date(),
  })
  try {    
    const savedNote = await note.save()    
    response.status(201).json(savedNote)  
  } catch(exception) {    
    next(exception)  
  }
})
```

The catch block simply calls the `next` function, which passes the request handling to the error handling middleware.

## Eliminating the try-catch

The express-async-errors library can help in removing try catch blocks. Install using:
```
npm i express-async-errors
```

Then introduce the library in *app.js*:
```
require('express-async-errors')
```

This allows us to turn a route using try catch blocks like this:
```
notesRouter.delete('/:id', async (request, response, next) => {
  try {
    await Note.findByIdAndRemove(request.params.id)
    response.status(204).end()
  } catch (exception) {
    next(exception)
  }
})
```

Into this:
```
notesRouter.delete('/:id', async (request, response) => {
  await Note.findByIdAndRemove(request.params.id)
  response.status(204).end()
})
```

The library handles everything under the hood. If an exception occurs in an *async* route, the execution is automatically passed to the error handling middleware.

## Optimizing the beforeEach function

Our current `beforeEach` function that sets up our tests looks like this:
```
beforeEach(async () => {
  await Note.deleteMany({})

  let noteObject = new Note(helper.initialNotes[0])
  await noteObject.save()

  noteObject = new Note(helper.initialNotes[1])
  await noteObject.save()
})
```

The function saves the first two notes from the `helper.initialNotes` array into the database with two separate operations. Here's an **attempt** to improve this function, which actually doesn't work:
```
beforeEach(async () => {
  await Note.deleteMany({})
  console.log('cleared')

  helper.initialNotes.forEach(async (note) => {
    let noteObject = new Note(note)
    await noteObject.save()
    console.log('saved')
  })
  console.log('done')
})
```

This won't work, and tests will run before the database is setup, because every iteration of the `forEach` loop generates an asynchronous operation, and `beforeEach` won't wait for them to finish executing. In other words, the `await` commands defined inside the `forEach` loop are not in the `beforeEach` function, but in separate functions that `beforeEach` will not wait for.

Since execution of tests begins immediately after `beforeEach` has finished, the tests begin before the database state is initialized.

One way to fix this is to wait for all of the asynchronous operations to finish executing with the Promise.all method:
```
beforeEach(async () => {
  await Note.deleteMany({})

  const noteObjects = helper.initialNotes
    .map(note => new Note(note))
  const promiseArray = noteObjects.map(note => note.save())
  await Promise.all(promiseArray)
})
```

The `noteObjects` variable is assigned to an array of Mongoose objects that are created with the `Note` constructor for each of the notes in the `helper.initialNotes` array. The next line of code creates a new array that *consists of promises*, that are created by calling the `save` method of each item in the `noteObjects` array. In other words, it is an array of promises for saving each of the items to the database.

The Promise.all method can be used for transforming an array of promises into a single promise that will be fulfilled once every promise in the array passed to it as a parameter is resolved. The last line of code `await Promise.all(promiseArray)` waits until every promise for saving a note is finished, meaning that the database has been initialized.

If we wait with the syntax `const results = await Promise.all(promiseArray)`, the operation will return an array that contains the resolved values for each promise in the `promiseArray`, and they appear in the same order as the promises in the array.

Promise.all executes the promises it receives in parallel. If the promises need to be executed in a particular order, it will not work. In situations like this, the operations can be executed in a for...of block, that guarantees a specific execution order:
```
beforeEach(async () => {
  await Note.deleteMany({})

  for (let note of helper.initialNotes) {
    let noteObject = new Note(note)
    await noteObject.save()
  }
})
```
