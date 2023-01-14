# End to end testing

So far, we have tested the backend as a whole on an API level using integration tests, and tested some frontend components using unit tests.

Next, we will look into one way to test the system as a whole using *End to End* (E2E) tests.

We can do E2E testing of a web application using a browser and a testing library. There are multiple libraries available. One example is Selenium, which can be used with almost any browser. Another browser option is so-called headless browsers, which are browsers with no graphical interface. For example, Chrome can be used in headless mode.

E2E tests are potentially the most useful category of tests because they test the system through the same interface as real users use.

They do have some drawbacks. Configuring E2E tests is more challenging than unit or integration tests. They also tend to be quite slow, and with a large system, their execution time can be minutes or even hours. E2E tests can also be flaky. Some tests might pass one time and fail another, even if the code does not change at all.

## Cypress

E2E library Cypress is a popular newer option. Cypress tests are run completely within the browser, as opposed to other libraries, which run the tests in a Node process, which is connected to the browser through an API.

We begin by installing Cypress to *the frontend* as a development dependency:
```
npm install --save-dev cypress
```

Then, add an npm-script to run it:
```
"cypress:open": "cypress open"
```

Unlike the frontend unit tests, Cypress tests can be in the frontend or backend repository, or even in their own separate repository.

The tests require the tested system to be running. Unlike our backend integration tests, Cypress tests *do not start* the system when they are run.

Let's add an npm script to the *backend* which starts it in test mode, or so that *NODE_ENV* is *test*:
```
"start:test": "cross-env NODE_ENV=test node index.js"
```

Note: to get Cypress working with WSL, follow the instructions on this page:
https://nickymeuleman.netlify.app/blog/gui-on-wsl2-cypress

When we first run Cypress, it creates a *cypress* directory. It contains an *e2e* subdirectory, where we will place our tests. Cypress creates a bunch of example tests for us in two subdirectories: the *e2e/1-getting-started* and the *e2e/2-advanced-examples* directory. We can delete both directories and make our test in the file *note_app.cy.js*:
```
describe('Note app', function() {
  it('front page can be opened', function() {
    cy.visit('http://localhost:3000')
    cy.contains('Notes')
    cy.contains('Note app, Department of Computer Science, University of Helsinki 2022')
  })
})
```

You can run the test from the opened Cypress window. It will open your browser and shows how the application behaves as the test is run.

Cypress tests also use *describe* blocks to group different test cases. Test cases are defined with the *it* method. Cypress borrowed these parts from the Mocha testing library it uses under the hood.

`cy.visit` and `cy.contains` are Cypress commands. `cy.visit` opens the web address given to it as a parameter in the browser used by the test. `cy.contains` searches for the string it received as a parameter from the page.

We could have declared the test using an arrow function. However, Mocha recommends that arrow functions are not used, because they might cause some issues in certain situations. Lambdas lexically bind `this`, and cannot access the Mocha context. If you do not need to use Mocha's context, lambdas should work, but should probably still be avoided to avoid needing to refactor in the future.

If `cy.contains` does not find the text it is searching for, the test does not pass.

## Writing to a form

Let's extend our tests so that the test tries to login to our application. We assume our backend contains a user with the username *kcmaxwell* and password *password123*.
```
describe('Note app', function() {
  beforeEach(function() {    
    cy.visit('http://localhost:3000')  
  })

  it('login form can be opened', function() {
    cy.contains('login').click()
  })

  it('user can log in', function() {
    cy.contains('login').click()
    cy.get('#username').type('kcmaxwell')    
    cy.get('#password').type('password123')    
    cy.get('#login-button').click()

    cy.contains('Kristopher Maxwell logged in')
})
```

The beforeEach function opens the page at *http://localhost:3000* where the application is. The first test searches for the login button by its text, and clicks the button with the command `cy.click`.

We added unique ids to the input fields in order to access them in the second test. We can write to them with the command `cy.type`. The last row ensures that the login was successful.

## Some things to note

If we search for a button by its text, `cy.contains` will return the first of them, or the one opening the login form. This will happen even if the button is not visible. To avoid name conflicts, we should use unique ids to access elements.

We can remove an ESLint error with the `cy` variable by installing *eslint-plugin-cypress* as a development dependency, and then changing our .eslintrc.js like so:
```
module.exports = {
    "env": {
        "browser": true,
        "es6": true,
        "jest/globals": true,
        "cypress/globals": true    },
    "extends": [ 
      // ...
    ],
    "parserOptions": {
      // ...
    },
    "plugins": [
        "react", "jest", "cypress"    ],
    "rules": {
      // ...
    }
}
```

## Testing new note form

Let's next add test methods to test the new note functionality:
```
describe('Note app', function() {
  // ..
  describe('when logged in', function() {    
    beforeEach(function() {      
        cy.contains('login').click()      
        cy.get('input:first').type('kcmaxwell')      
        cy.get('input:last').type('password123')      
        cy.get('#login-button').click()    
    })
    
    it('a new note can be created', function() {      
        cy.contains('new note').click()      
        cy.get('input').type('a note created by cypress')      
        cy.contains('save').click()      
        cy.contains('a note created by cypress')    
    })  
  })
})
```

The test has been defined in its own *describe* block. Only logged-in users can create new notes, so we added logging in to the application to a *beforeEach* block.

The test trusts that when creating a new note, the page contains only one input, so it searches for it using `cy.get('input')`. If the page contained more inputs, the test would break, so it would be best to change this to using a unique id to access the input.

Cypress runs tests in the order they are in the code. Each test *starts from zero* as far as the browser is concerned. All changes to the browser's state are reversed after each test.

## Controlling the state of the database

If the tests need to be able to modify the server's database, the situation immediately becomes more complicated. Ideally, the server's database should be the same each time we run the tests, so our tests can be reliably and easily repeatable.

As with unit and integration tests, with E2E tests, it is best to empty the database and possibly format it before the tests are run. The challenge with E2E tests is that they do not have access to the database.

The solution is to create API endpoints for the backend tests. We can empty the database using these endpoints. Let's create a new router for the tests:
```
const testingRouter = require('express').Router()
const Note = require('../models/note')
const User = require('../models/user')

testingRouter.post('/reset', async (request, response) => {
  await Note.deleteMany({})
  await User.deleteMany({})

  response.status(204).end()
})

module.exports = testingRouter
```

Then, we add it to the backend *only if the application is run in test mode*:
```
app.use('/api/login', loginRouter)
app.use('/api/users', usersRouter)
app.use('/api/notes', notesRouter)

if (process.env.NODE_ENV === 'test') {  
    const testingRouter = require('./controllers/testing')  
    app.use('/api/testing', testingRouter)
}

app.use(middleware.unknownEndpoint)
app.use(middleware.errorHandler)

module.exports = app
```

After the changes, an HTTP POST request to the */api/testing/reset* endpoint empties the database. Make sure your backend is running in test mode by starting it with the command you added at the start of this section, `npm run start:test`.

Next, we will change the *beforeEach* block so that it empties the server's database before tests are run.

Currently, it is not possible to add new users through the frontend's UI, so we add a new user to the backend from the beforeEach block.
```
beforeEach(function() {
    cy.request('POST', 'http://localhost:3001/api/testing/reset')   
    const user = {      
        name: 'Kristopher Maxwell',      
        username: 'kcmaxwell',      
        password: 'password123'    
    }    
    cy.request('POST', 'http://localhost:3001/api/users/', user)     
    cy.visit('http://localhost:3000')
})
```

During the formatting, the test does HTTP requests to the backend with `cy.request`.

Unlike earlier, now the testing starts with the backend in the same state every time. The backend will contain one user and no notes.

## Failed login test

Let's make a test to ensure that a login attempt fails if the password is wrong.

Cypress will run all tests each time by default, and as the number of tests increases, it starts to become quite time-consuming. When developing a new test, or when debugging a broken test, we can define the test with *it.only* instead of *it*, so that Cypress will only run the required test. When the test is working, we can remove *.only*.

Here is the finished test:
```
it('login fails with wrong password', function() {
  cy.contains('login').click()
  cy.get('#username').type('kcmaxwell')
  cy.get('#password').type('wrong')
  cy.get('#login-button').click()

  cy.get('.error')
    .should('contain', 'wrong credentials')
    .and('have.css', 'color', 'rgb(255, 0, 0)')
    .and('have.css', 'border-style', 'solid')

  cy.get('html').should('not.contain', 'Kristopher Maxwell logged in')})
```

We could use `cy.contains('text content')` instead of `cy.should`, but should allows for more diverse tests than contains, which only works based on text content.

First, we use `cy.get` to search for a component with the CSS class *error*. Then, we check that the error message can be found from this component. We also check if the error message element has the correct color and border. We do this by chaining them using `and`.

We finish by checking that the application does not render a success message.

Note: some CSS properties behave differently on different browsers, like Firefox. Tests that involve, for example, border-style, border-radius, and padding will pass in Chrome, but fail in Firefox.

## Bypassing the UI

The Cypress documentation gives us the following advice: fully test the login flow, but only once. So instead of logging in a user using the form in the *beforeEach* block, Cypress recommends that we bypass the UI and do an HTTP request to the backend to log in. The reason for this is that logging in with an HTTP request is much faster than filling out a form.

In our case, our application saves the user details to localStorage when logging in, and Cypress can handle that as well. The code is the following:
```
describe('when logged in', function() {
  beforeEach(function() {
    cy.request('POST', 'http://localhost:3001/api/login', {      
        username: 'kcmaxwell', password: 'password123'    
    }).then(response => {      
        localStorage.setItem('loggedNoteappUser', JSON.stringify(response.body))      
        cy.visit('http://localhost:3000')    
    })  
  })

  it('a new note can be created', function() {
    // ...
  })

  // ...
})
```

We can access the response to a `cy.request` with the `then` method. Under the hood, `cy.request`, like all Cypress commands, is a promise. The callback function saves the details of a logged-in user to localStorage, and reloads the page. Now, there is no difference to a user logging in with the login form.

If and when we write new tests to our application, we have to use the login code in multiple places. We can make it a custom command.

Custom commands are declared in *cypress/support/commands.js*. The code for logging in is as follows:
```
Cypress.Commands.add('login', ({ username, password }) => {
  cy.request('POST', 'http://localhost:3001/api/login', {
    username, password
  }).then(({ body }) => {
    localStorage.setItem('loggedNoteappUser', JSON.stringify(body))
    cy.visit('http://localhost:3000')
  })
})
```

Our new custom command can be used as follows:
```
describe('when logged in', function() {
  beforeEach(function() {
    cy.login({ username: 'kcmaxwell', password: 'password123' })  })

  it('a new note can be created', function() {
    // ...
  })

  // ...
})
```

We can also make a custom command for making a new note. The command will make a new note with an HTTP POST request:
```
Cypress.Commands.add('createNote', ({ content, important }) => {
  cy.request({
    url: 'http://localhost:3001/api/notes',
    method: 'POST',
    body: { content, important },
    headers: {
      'Authorization': `bearer ${JSON.parse(localStorage.getItem('loggedNoteappUser')).token}`
    }
  })

  cy.visit('http://localhost:3000')
})
```

The command expects the user to be logged in, and the user's details to be saved to localStorage.

## Changing the importance of a note

We'll change the formatting block of this test so that it creates 3 notes instead of one, using our new custom command:
```

describe('when logged in', function() {
  describe('and several notes exist', function () {
    beforeEach(function () {
      cy.createNote({ content: 'first note', important: false })      
      cy.createNote({ content: 'second note', important: false })      
      cy.createNote({ content: 'third note', important: false })    
    })

    it('one of those can be made important', function () {
      cy.contains('second note')
        .contains('make important')
        .click()

      cy.contains('second note')
        .contains('make not important')
    })
  })
})
```

Notice that we chained the `contains` function after getting the note with the contents 'second note'. If we did not, it would capture the first element it finds with the text 'make important', which would be the wrong button.

If we change the Note component and put the contents of the note into a span, the test will break. `cy.contains('second note')` will return the span, but the button is not in it.

One way to fix this is the following:
```
it('one of those can be made important', function () {
  cy.contains('second note').parent().find('button').click()
  cy.contains('second note').parent().find('button')
    .should('contain', 'make not important')
})
```

In the first line, we use the `parent` command to access the parent element of the element containing *second note*, and find the button from within it. Then, we click the button and check that the text on it changes.

Note that we use the command `find` to search for the button. We cannot use `cy.get` here, because it always searches from the *whole page*, and would return all the buttons on the page.

To remove some of the repeated code, we can use the `as` command:
```
it('one of those can be made important', function () {
  cy.contains('second note').parent().find('button').as('theButton')
  cy.get('@theButton').click()
  cy.get('@theButton').should('contain', 'make not important')
})
```

The first line finds the right button and uses `as` to save it as *theButton*. The following lines can use the named element with *cy.get('@theButton')*.

## Running and debugging the tests

Finally, some notes on how Cypress works and debugging your tests.

The form of the Cypress tests gives the impression that the tests are normal JavaScript code. Trying something like the code below would not work, however:
```
const button = cy.contains('login')
button.click()
debugger() 
cy.contains('logout').click()
```

When Cypress runs a test, it adds each `cy` command to an execution queue. When the code of the test method has been executed, Cypress will execute each command in the queue one by one.

Cypress commands always return `undefined`, so `button.click()` in the above code would cause an error. An attempt to start the debugger would not stop the code between executing the commands, but before any commands have been executed.

Cypress commands are *like promises*, so if we want to access their return values, we have to do it using the `then` command. For example, the following test would print the number of buttons in the application, then click the first button:
```
it('then example', function() {
  cy.get('button').then( buttons => {
    console.log('number of buttons', buttons.length)
    cy.wrap(buttons[0]).click()
  })
})
```

Stopping the test execution with the debugger is possible. The debugger starts only if Cypress test runner's developer console is open.

The developer console is all sorts of useful when debugging your tests. You can see the HTTP requests done by the tests on the Network tab, and the console tab will show you information about your tests.

You can also run Cypress tests from the command line. We just have to add an npm script for it:
```
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "server": "json-server -p3001 --watch db.json",
    "cypress:open": "cypress open",
    "test:e2e": "cypress run"  
},
```

Now, we can run our tests from the command line with the command `npm run test:e2e`.

Videos of the test execution will be saved to *cypress/videos/*, so you should probably git ignore this directory.

