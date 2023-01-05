# Saving data to MongoDB

MongoDB is a document database, as opposed to a relational database. Document databases differ from relational databases in how they organize data, as well as in the query languages they support. Document databases are usually categorized under the NoSQL umbrella term.

To connect to MongoDB Atlas, first, under security, create a user and password, and make sure your IP is under Network Access.

Next, click connect on your cluster, and choose "Connect your application". Copy the connection string into a .env file, and be sure to replace the username and password with what you created in the first step.

You can change the name of the database by adding the desired name to the string right before "?retryWrites".

We will use the Mongoose library to connect to MongoDB. Mongoose could be described as an *object document mapper* (ODM), and saving JavaScript objects as Mongo documents is straightforward with this library.

Install mongoose with npm: `npm install mongoose`

Here is an example file that takes command line parameters to add a note to a collection on MongoDB using mongoose:
```
const mongoose = require('mongoose')

if (process.argv.length < 3) {
  console.log('Please provide the password as an argument: node mongo.js <password>')
  process.exit(1)
}

const password = process.argv[2]

const url = `mongodb+srv://notes-app-full:${password}@cluster1.lvvbt.mongodb.net/noteApp?retryWrites=true&w=majority`

const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  important: Boolean,
})

const Note = mongoose.model('Note', noteSchema)

mongoose
  .connect(url)
  .then((result) => {
    console.log('connected')

    const note = new Note({
      content: 'HTML is Easy',
      date: new Date(),
      important: true,
    })

    return note.save()
  })
  .then(() => {
    console.log('note saved!')
    return mongoose.connection.close()
  })
  .catch((err) => console.log(err))
```

## Schema

After establishing the connection to the database, we define the schema for a note and the matching model:
```
const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  important: Boolean,
})

const Note = mongoose.model('Note', noteSchema)
```

First, we define the schema of a note that is stored in the `noteSchema` variable. The schema tells Mongoose how the note objects are to be stored in the database.

In the `Note` model definition, the first *"Note"* parameter is the singular name of the model. The name of the collection will be the lowercase plural *notes*, because the Mongoose convention is to automatically name collections as the plural when the schema refers to them in the singular.

Document databases like Mongo are *schemaless*, meaning that the database itself does not care about the structure of the data that is stored in the database. It is possible to store documents with completely different fields in the same collection.

The idea behind Mongoose is that the data stored in the database is given a *schema at the level of the application* that defines the shape of the documents stored in any given collection.

## Creating and saving objects

Next, the application creates a new note object with the help of the *Note* model:
```
const note = new Note({
    content: 'HTML is Easy',
    date: new Date(),
    important: false,
})
```

Models are so-called *constructor functions* that create new JavaScript objects based on the provided parameters. Since the objects are created with the model's constructor function, they have all the properties of the model, which include methods for saving the object to the database.

Saving the object to the database happens with the appropriately named `save` method, which can be provided with an event handler with the `then` method:
```
note.save().then(result => {
  console.log('note saved!')
  mongoose.connection.close()
})
```

When the object is saved to the database, the event handler provided to `then` gets called. The event handler closes the database connection with the command `mongoose.connection.close()`. If the connection is not closed, the program will never finish its execution.

The result of the save operation is in the `result` parameter of the event handler.

## Fetching objects from the database

This code will retrieve all notes from the collection:
```
Note.find({}).then(result => {
  result.forEach(note => {
    console.log(note)
  })
  mongoose.connection.close()
})
```

The objects are retrieved from the database with the find method of the `Note` model. The parameter of the method is an object expressing search conditions. Since the parameter is an empty object `{}`, we get all of the notes stored in the `notes` collection.

The search conditions adhere to the Mongo search query syntax. More info on that syntax can be found here:
https://www.mongodb.com/docs/manual/reference/operator/

## Connecting the backend to a database

We don't want to return the mongo versioning field *__v* to the frontend. The frontend is also expecting an *id* field, not an *_id* field that mongo provides.

One way to format the objects returned by Mongoose is to modify the `toJSON` method of the schema, which is used on all instances of the models produced with that schema:
```
noteSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
  }
})
```

Even though the *_id* property of Mongoose objects looks like a string, it is in factr an object. The `toJSON` method we defined transforms it into a string just to be safe. When we call something like `response.json(notes)` on objects returned by Mongo, the `toJSON` method of each object will be automatically called by the JSON.stringify method.

## Database configuration into its own module

Defining Node modules differs slightly from the way of defining ES6 modules used in part 2:
```
module.exports = mongoose.model('Note', noteSchema)
```

The public interface of the module is defined by setting a value to the `module.exports` variable. Above, we set the value to be the *Note* model. The other things defined inside of the module will not be accessible or visible to users of the module.

To import the module if it is in the relative directory */models/note.js*, we do this:
```
const Note = require('./models/note')
```

We should not hardcode the address of the database into the code, so we will use a `MONGODB_URI` environment variable instead. We can do this using the dotenv library, installed using `npm install dotenv`.

We create a *.env* file at the root of the project, and add our environment variables to it:
```
MONGODB_URI=mongodb+srv://fullstack:<password>@cluster0.o1opl.mongodb.net/noteApp?retryWrites=true&w=majority
PORT=3001
```

**This .env file should be gitignored right away.**

The environment variables defined in the *.env* file can be taken into use with the expression `require('dotenv').config()`, and you can reference them like normal environment variables, eg. `process.env.MONGODB_URI`.

It's important that *dotenv* gets imported before the *note* model is imported. This ensures that the environment variables from the *.env* file are available globally before the code from the other modules is imported.

Because GitHub is not used with Fly.io, the .env file gets sent to the Fly.io servers when the app is deployed. Because of this, the env variables defined in the file will be available there.

A better option is to prevent .env from being copied to Fly.io by creating the file *.dockerignore* to the root directory, containing the line `.env`. You can then set the environment values like so:
```
fly secrets set MONGODB_URI='mongodb+srv://fullstack:<password>@cluster0.o1opl.mongodb.net/noteApp?retryWrites=true&w=majority'
```

## Using database in route handlers

Using Mongoose's findById method, fetching an individual note gets changed into the following:
```
app.get('/api/notes/:id', (request, response) => {
  Note.findById(request.params.id).then(note => {
    response.json(note)
  })
})
```

## Verifying frontend and backend integration

When the backend gets expanded, it's a good idea to test the backend first. Only once everything has been verified to work in the backend is it a good idea to test that the frontend works with the backend. It is highly inefficient to test things exclusively through the frontend.

It's probably a good idea to integrate the frontend and backend one functionality at a time. First, we could implement fetching all notes from the database, test it through the backend endpoint in the browser, then verify that the frontend works with the new backend. Once everything seems to be working, we would move on to the next feature.

Once we introduce a database into the mix, it is useful to inspect the state persisted in the database, eg. from the control panel in MongoDB Atlas. Often, small helper Node programs can be very helpful during development.

## Error handling

If we try to visit the URL of a note with an id that does not exist, then the response will be `null`.

The following code will have the server respond with the HTTP status code 404 if the id doesn't exist. It also adds a `catch` block to handle cases where the promise returned by the `findById` method is *rejected*:
```
app.get('/api/notes/:id', (request, response) => {
  Note.findById(request.params.id)
    .then(note => {
      if (note) {        
        response.json(note)      
        } else {        
            response.status(404).end()      
            }    
        })
    .catch(error => {      
        console.log(error)
        response.status(400).send({ error: 'malformatted id' })  
    })
})
```

If no matching object is found in the database, the value of `note` will be null, and the else block is executed. This results in a response with the status code *404 not found*. If a promise returned by `findById` is rejected, the response will have the status code *400*. If we try to fetch a note with the wrong kind of id, meaning an id that doesn't match the mongo identifier format, we receive an error, which will trigger the catch block and send the error "malformatted id".

## Moving error handling into middleware

There are cases where it is better to implement all error handling in a single place. This is particularly useful if we want to report data related to errors to an external error-tracking system like Sentry later on.

Let's modify the code from above so that it passes the error forward with the `next` function. The next function is passed to the handler as the third parameter:
```
app.get('/api/notes/:id', (request, response, next) => {  
  Note.findById(request.params.id)
    .then(note => {
      if (note) {
        response.json(note)
      } else {
        response.status(404).end()
      }
    })
    .catch(error => next(error))})
```

The error is given to the `next` function as a parameter. If `next` was called without a parameter, then the execution would simply move onto the next route or middleware. If the `next` function is called with a parameter, then the execution will continue to the *error handler middleware*.

Express error handlers are middleware that are defined with a function that accepts *four parameters*. The error handler for the notes app looks like this:
```
const errorHandler = (error, request, response, next) => {
  console.error(error.message)

  if (error.name === 'CastError') {
    return response.status(400).send({ error: 'malformatted id' })
  } 

  next(error)
}

// this has to be the last loaded middleware.
app.use(errorHandler)
```

The error handler checks if the error is a *CastError* exception, in which case we know that the error was caused by an invalid object id for Mongo. In this situation, the error handler will send a response to the browser with the response object passed as a parameter. In all other error situations, the middleware passes the error forward to the default Express error handler.

Note that the error-handling middleware has to be the **last loaded middleware.**

## The order of middleware loading

The execution order of middleware is the same as the order that they are loaded into express with the `app.use` function. For this reason, it is important to be careful when defining middleware.

The correct order is the following:
```
app.use(express.static('build'))
app.use(express.json())
app.use(requestLogger)

app.post('/api/notes', (request, response) => {
  const body = request.body
  // ...
})

const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

// handler of requests with unknown endpoint
app.use(unknownEndpoint)

const errorHandler = (error, request, response, next) => {
  // ...
}

// handler of requests with result to errors
app.use(errorHandler)
```

The json-parser middleware should be among the very first middleware loaded into Express, otherwise `request.body` will be undefined in route handlers.

It's also important that the middleware for handling unsupported routes is next to the last middleware that is loaded into Express, just before the error handler, as no routes or middleware will be called after the *404 unknown endpoint* status. The only exception to this is the error handler, which needs to come at the very end.

## Other operations

The easiest way to delete a note from the database is with the findByIdAndRemove method:
```
app.delete('/api/notes/:id', (request, response, next) => {
  Note.findByIdAndRemove(request.params.id)
    .then(result => {
      response.status(204).end()
    })
    .catch(error => next(error))
})
```

In both successful cases of deleting a resource, the backend responds with the status code *204 no content*. The two different cases are deleting a note that exists, and deleting a note that does not exist in the database. The `result` callback parameter could be used for checking if a resource was actually deleted. Any exception that occurs is passed onto the error handler.

PUT requests can be handled using the findByIdAndUpdate method:
```
app.put('/api/notes/:id', (request, response, next) => {
  const body = request.body

  const note = {
    content: body.content,
    important: body.important,
  }

  Note.findByIdAndUpdate(request.params.id, note, { new: true })
    .then(updatedNote => {
      response.json(updatedNote)
    })
    .catch(error => next(error))
})
```

Notice that the `findByIdAndUpdate` method receives a regular JavaScript object as its parameter, and not a new note object created with the `Note` constructor function.

There is one important detail regarding the use of the `findByIdAndUpdate` method. By default, the `updatedNote` parameter of the event handler receives the original document without the modifications. We added the optional `{ new: true }` parameter, which will cause our event handler to be called with the new modified document instead of the original.
