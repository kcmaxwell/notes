# Structure of backend, intro to testing

## Project structure

We will modify the structure of our notes project to adhere to Node.js best practices. Here is the directory structure we will use:
```
├── index.js
├── app.js
├── build
│   └── ...
├── controllers
│   └── notes.js
├── models
│   └── note.js
├── package-lock.json
├── package.json
├── utils
│   ├── config.js
│   ├── logger.js
│   └── middleware.js  
```

We will separate printing to the console to its own module *utils/logger.js*:
```
const info = (...params) => {
  console.log(...params)
}

const error = (...params) => {
  console.error(...params)
}

module.exports = {
  info, error
}
```

info prints normal log messages, and error prints error messages.

Extracting logging into its own module is a good idea, as it makes changing to writing logs to a file or sending them to an external logging service much easier, since we only need to make changes in one place.

*index.js* is simplified into just imports and starting the server:
```
const app = require('./app') // the actual Express application
const http = require('http')
const config = require('./utils/config')
const logger = require('./utils/logger')

const server = http.createServer(app)

server.listen(config.PORT, () => {
  logger.info(`Server running on port ${config.PORT}`)
})
```

The handling of environment variables is extracted into a separate *utils/config.js* file:
```
require('dotenv').config()

const PORT = process.env.PORT
const MONGODB_URI = process.env.MONGODB_URI

module.exports = {
  MONGODB_URI,
  PORT
}
```

The other parts of the application can access the environment variables by importing the configuration module:
```
const config = require('./utils/config')

logger.info(`Server running on port ${config.PORT}`)
```

Route handlers have also been moved into a dedicated module. The event handlers of routes are commonly referred to as *controllers*, and for this reason, we have created a new *controllers* directory. All of the routes related to notes are now in the *notes.js* module in the *controllers* directory.

The contents of *notes.js* are the following:
```
const notesRouter = require('express').Router()

// routes
// eg: notesRouter.get('/:id', ...)

module.exports = notesRouter
```

A router object, like *notesRouter* above, is an isolated instance of middleware and routes. You can think of it as a "mini-application", capable only of performing middleware and routing functions. Every Express application has a built-in app router.

One thing to note is that the URLs used are shorter than before. This is because *app.js* defines the notesRouter to be used if the URL starts with */api/notes*, so routes in *notes.js* must be relative to that URL. This is the import of *notes.js*:
```
const notesRouter = require('./controllers/notes')
app.use('/api/notes', notesRouter)
```

Our custom middleware has been moved to a new *utils/middleware.js* module.

The responsibility of establishing the connection to the database has been given to *app.js* The *note.js* file in the *models* directory only defines the Mongoose schema for notes.

There is no strict directory structure or file naming convention required for Express applications. Our current structure simply follows some of the best practices you can come across on the internet.

## Note on exports

The file *utils/logger.js* does the export as follows:
```
const info = (...params) => {
  console.log(...params)
}

const error = (...params) => {
  console.error(...params)
}

module.exports = {  info, error
```

The file exports *an object* that has two fields, both of which are functions. The functions can be used in two different ways. The first option is to require the whole object and refer to functions through the object using the dot notation:
```
const logger = require('./utils/logger')

logger.info('message')

logger.error('error message')
```

The other option is to destructure the functions to their own variables in the *require* statement:
```
const { info, error } = require('./utils/logger')

info('message')
error('error message')
```

The latter way may be preferable if only a small portion of the exported functions are used in a file.

## Testing Node applications

Here is a small file we will add as *utils/for_testing.js* to practice writing tests:
```
const reverse = (string) => {
  return string
    .split('')
    .reverse()
    .join('')
}

const average = (array) => {
  const reducer = (sum, item) => {
    return sum + item
  }

  return array.reduce(reducer, 0) / array.length
}

module.exports = {
  reverse,
  average,
}
```

We will be using the testing library jest, which resembles a previously popular testing library Mocha.

Since tests are only executed during the development of our application, we will install *jest* as a development dependency with the command: `npm install --save-dev jest`

We can add a *test* npm script to execute tests with Jest and report about the test execution with the *verbose* style: `"test": "jest --verbose"

Jest also requires us to specify that the execution environment is Node. This can be done by adding the following to the end of *package.json*:
```
{
 //...
 "jest": {
   "testEnvironment": "node"
 }
}
```

Alternatively, Jest can look for a configuration file with the default name *jest.config.js*, where we can define the execution environment like this:
```
module.exports = {
  testEnvironment: 'node',
}
```

Let's create a separate directory for our tests called *tests* and create a new file called *reverse.test.js* with the following contents:
```
const reverse = require('../utils/for_testing').reverse

test('reverse of a', () => {
  const result = reverse('a')

  expect(result).toBe('a')
})

test('reverse of react', () => {
  const result = reverse('react')

  expect(result).toBe('tcaer')
})

test('reverse of releveler', () => {
  const result = reverse('releveler')

  expect(result).toBe('releveler')
})
```

ESLint will complain about the `test` and `expect` commands in our test file, since the configuration does not allow *globals*. Simply add `"jest": true` to the *env* property in the *.eslintrc.js* file to fix this.

In the first row, the test file imports the function to be tested and assigns it to a variable called `reverse`. Individual test cases are defined with the `test` function. The first parameter of the function is the test description as a string. The second parameter is a *function* that defines the functionality for the test case.

For example, with this test case:
```
test('reverse of react', () => {
  const result = reverse('react')

  expect(result).toBe('tcaer')
})
```

First, we execute the code to be tested, meaning that we generate a reverse for the string *react*. Next, we verify the results with the expect function. Expect wraps the resulting value into an object that offers a collection of *matcher* functions that can be used for verifying the correctness of the result. Since in this test case we are comparing two strings, we can use the toBe matcher.

Jest expects by default that the names of test files contain *.test*. We will follow the convention of naming our test files with the extension *.test.js*.

Describe blocks can be used for grouping tests into logical collections. The test output of Jest also uses the name of the describe block. We can create one like so:
```
describe('average', () => {
  // tests
})
```

We will see later on that *describe* blocks are necessary when we want to run some shared setup or teardown operations for a group of tests.
