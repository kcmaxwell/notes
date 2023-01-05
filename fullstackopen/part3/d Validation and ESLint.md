# Validation and ESLint

There are usually constraints that we want to apply to the data that is stored in our application's database. Our application shouldn't accept notes that have a missing or empty *content* property. We do check for validity of the note in the route handler. If the note does not have the *content* property, we respond to the request with the status code *400 bad request*.

One smarter way of validating the format of the data before it is stored in the database is to use the validation functionality available in Mongoose.

We can define specific validation rules for each field in the schema:
```
const noteSchema = new mongoose.Schema({
  content: {    
    type: String,    
    minLength: 5,    
    required: true  
  },  
  date: {     
    type: Date,    
    required: true  
  },  
  important: Boolean
})
```

The *content* field is now required to be at least five characters long. The *date* field is set as required, meaning that it cannot be missing. The same constraint is also applied to the *content* field, since the minimum length constraint allows the field to be missing.

The *minLength* and *required* validators are built-in and provided by Mongoose. The Mongoose custom validator functionality allows us to create new validators if none of the built-in ondes cover our needs.

If we try to store an object in the database that breaks one of the constraints, the operation will throw an exception. Make sure the handler for creating new notes sends the error to the error handler via `next`.

Let's also expand the error handler to deal with these validation errors:
```
const errorHandler = (error, request, response, next) => {
  console.error(error.message)

  if (error.name === 'CastError') {
    return response.status(400).send({ error: 'malformatted id' })
  } else if (error.name === 'ValidationError') {    
    return response.status(400).json({ error: error.message })  
  }

  next(error)
}
```

Note that **validations are not run by default when findOneAndUpdate is executed.** This means PUT routes will not use your given validators.

To fix this, you need to add the option *runValidators* set to true in the options for findByIDAndUpdate:
```
app.put('/api/notes/:id', (request, response, next) => {
  const { content, important } = request.body

  Note.findByIdAndUpdate(
    request.params.id, 
    { content, important },    
    { new: true, runValidators: true, context: 'query' }  
  ) 
    .then(updatedNote => {
      response.json(updatedNote)
    })
    .catch(error => next(error))
})
```

## Lint

Generically, lint or a linter is any tool that detects and flags errors in programming languages, including stylistic errors. The term lint-like behavior is sometimes applied to the process of flagging suspicious language usage. Lint-like tools generally perform static analysis of source code.

In the JavaScript universe, the current leading tool for static analysis aka "linting" is ESlint.

To install ESLint, we can use npm: `npm install eslint --save-dev`

After this, we can initialize a default ESlint configuration with the command: `npx eslint --init`

The configuration will be saved in the `.eslintrc.js` file.

Inspecting and validating a file like index.js can be done with the following command:
```
npx eslint index.js
```

It is recommended to create a separate npm script for linting:
```
{
  // ...
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    // ...
    "lint": "eslint ."  },
  // ...
}
```

With the above script, files in the build directory will be checked when the command is run. We do not want this to happen, and we can accomplish this by creating a .eslintignore file in the project's root with the line: `build`

A better alternative to executing the linter from the command line is to configure a *eslint-plugin* to the editor that runs the linter continuously.

We can add extra rules by editing the .eslinrc.js file. For example, the eqeqeq rule warns us if equality is checked with anything but the triple equals operator. We add it under the *rules* field in the configuration file:
```
{
  // ...
  'rules': {
    // ...
   'eqeqeq': 'error',
  },
}
```

To disable a rule, we define its value as 0. For example: `'no-console': 0`
