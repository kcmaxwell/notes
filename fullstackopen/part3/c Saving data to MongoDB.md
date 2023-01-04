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

