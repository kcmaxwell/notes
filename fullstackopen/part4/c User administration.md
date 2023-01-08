# User administration

We want to add user authentication and authorization to our application. Users should be stored in the database, and every note should be linked to the user who created it. Deleting and editing a note should only be allowed for the user who created it.

Let's start by adding information about users to the database. There is a one-to-many relationship between the user and notes.
```mermaid
classDiagram
    User "1" --> "*" Note
```

If we were working with a relational database, the implementation would be straightforward. Both resources would have their separate database tables, and the id of the user who created a note would be stored in the notes table as a foreign key.

When working with document databases, the situation is a bit different, as there are many different ways of modeling the situation.

The existing solution saves every note in the *notes collection* in the database. If we do not want to change this existing collection, then the natural choice is to save users in their own collection, *users* for example.

Like with all document databases, we can use object IDs in Mongo to reference documents in other collections. This is similar to using foreign keys in relational databases.

Traditionally, document databases like Mongo do not support *join queries* that are available in relational databases, used for aggregating data from multiple tables. However, starting from version 3.2, Mongo has supported lookup aggregation queries. We will not be looking at this functionality.

If we need functionality similar to join queries, we will implement it in our application code by making multiple queries. In certain situations, Mongoose can take care of joining and aggregating data, which gives the appearance of a join query. However, even in these situations, Mongoose makes multiple queries to the database in the background.

## References across collections

If we were using a relational database, the note would contain a *reference key* to the user who created it. In document databases, we can do the same thing.

Let's assume that the *users* collection contains two users:
```
[
  {
    username: 'mluukkai',
    _id: 123456,
  },
  {
    username: 'hellas',
    _id: 141414,
  },
];
```

The *notes* collection contains three notes that all have a *user* field that references a user in the *users* collection:
```
[
  {
    content: 'HTML is easy',
    important: false,
    _id: 221212,
    user: 123456,
  },
  {
    content: 'The most important operations of HTTP protocol are GET and POST',
    important: true,
    _id: 221255,
    user: 123456,
  },
  {
    content: 'A proper dinosaur codes with Java',
    important: false,
    _id: 221244,
    user: 141414,
  },
]
```

Document databases do not demand the foreign key to be stored in the note resources. It could also be stored in the users collection, or even both:
```
[
  {
    username: 'mluukkai',
    _id: 123456,
    notes: [221212, 221255],
  },
  {
    username: 'hellas',
    _id: 141414,
    notes: [221244],
  },
]
```

Since users can have many notes, the related ids are stored in an array in the *notes* field.

Document databases also offer a radically different way of organizing the data: in some situations, it might be beneficial to nest the entire notes array as a part of the documents in the users collection:
```
[
  {
    username: 'mluukkai',
    _id: 123456,
    notes: [
      {
        content: 'HTML is easy',
        important: false,
      },
      {
        content: 'The most important operations of HTTP protocol are GET and POST',
        important: true,
      },
    ],
  },
  {
    username: 'hellas',
    _id: 141414,
    notes: [
      {
        content:
          'A proper dinosaur codes with Java',
        important: false,
      },
    ],
  },
]
```

In this schema, notes would be tightly nested under users, and the database would not generate ids for them.

The structure and schema of the database are not as self-evident as it was with relational databases. The chosen schema must support the use cases of the application the best. This is not a simple design decision to make, as all use cases of the applications are not known when the design decision is made.

## Mongoose schema for users

In this case, we will store the ids of the notes created by the user in the user document. The model for representing a user will be in the *models/user.js* file:
```
const mongoose = require('mongoose')

const userSchema = new mongoose.Schema({
  username: String,
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})

userSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
    // the passwordHash should not be revealed
    delete returnedObject.passwordHash
  }
})

const User = mongoose.model('User', userSchema)

module.exports = User
```

The ids of the notes are stored within the user document as an array of Mongo ids. The type of the field is *ObjectId* that references *note*-style documents. Mongo does not inherently know that this is a field that references notes, the syntax is purely related to and defined by Mongoose.

Here is an expanded schema for notes in *models/note.js* so that the note contains information about the user who created it:
```
const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    required: true,
    minlength: 5
  },
  date: Date,
  important: Boolean,
  user: {    
    type: mongoose.Schema.Types.ObjectId,    
    ref: 'User'  
  }
})
```

In stark contrast to the conventions of relational databases, *references are now stored in both documents*: the note references the user who created it, and the user has an array of references to all of the notes created by them.

## Creating users

Let's implement a route for creating new users. Users have a unique *username*, a *name*, and something called a *passwordHash*. The password hash is the output of a one-way hash function applied to the user's password. It is never wise to store unencrypted plain text passwords in the database.

Let's install the bcrypt package for generating the password hashes:
```
npm install bcrypt
```

Creating new users is done by making an HTTP POST request to the *users* path. Let's define a separate *router* for dealing with users in a new *controllers/users.js* file. We need to take the router into use in the *app.js* file, so that it handles requests made to the */api/users* file:
```
const usersRouter = require('./controllers/users')

// ...

app.use('/api/users', usersRouter)
```

The contents of *controlles/users.js* are as follows:
```
const bcrypt = require('bcrypt')
const usersRouter = require('express').Router()
const User = require('../models/user')

usersRouter.post('/', async (request, response) => {
  const { username, name, password } = request.body

  const saltRounds = 10
  const passwordHash = await bcrypt.hash(password, saltRounds)

  const user = new User({
    username,
    name,
    passwordHash,
  })

  const savedUser = await user.save()

  response.status(201).json(savedUser)
})

module.exports = usersRouter
```

The password sent in the request is *not* stored in the database. We store the *hash* of the password that is generated with the `bcrypt.hash` function.

An initial test for this route could look like this:
```
const bcrypt = require('bcrypt')
const User = require('../models/user')

//...

describe('when there is initially one user in db', () => {
  beforeEach(async () => {
    await User.deleteMany({})

    const passwordHash = await bcrypt.hash('sekret', 10)
    const user = new User({ username: 'root', passwordHash })

    await user.save()
  })

  test('creation succeeds with a fresh username', async () => {
    const usersAtStart = await helper.usersInDb()

    const newUser = {
      username: 'mluukkai',
      name: 'Matti Luukkainen',
      password: 'salainen',
    }

    await api
      .post('/api/users')
      .send(newUser)
      .expect(201)
      .expect('Content-Type', /application\/json/)

    const usersAtEnd = await helper.usersInDb()
    expect(usersAtEnd).toHaveLength(usersAtStart.length + 1)

    const usernames = usersAtEnd.map(u => u.username)
    expect(usernames).toContain(newUser.username)
  })
})
```

The tests use the *usersInDb()* helper function that we implemented in the *tests/test_helper.js* file. The function is used to help verify the state of the database after a user is created:
```
const User = require('../models/user')

// ...

const usersInDb = async () => {
  const users = await User.find({})
  return users.map(u => u.toJSON())
}

module.exports = {
  initialNotes,
  nonExistingId,
  notesInDb,
  usersInDb,
}
```

Mongoose does not have a built-in validator for checking the uniqueness of a field. We could use a ready-made solution like mongoose-unique-validator, but it currently does not work with Mongoose version 6.x, so we have to implement the uniqueness check by ourselves in the controller:
```
usersRouter.post('/', async (request, response) => {
  const { username, name, password } = request.body

  const existingUser = await User.findOne({ username })  if (existingUser) {    return response.status(400).json({      error: 'username must be unique'    })  }
  const saltRounds = 10
  const passwordHash = await bcrypt.hash(password, saltRounds)

  const user = new User({
    username,
    name,
    passwordHash,
  })

  const savedUser = await user.save()

  response.status(201).json(savedUser)
})
```

## Creating a new note with users

We now have to update the code for creating a new note, to assign the user that created it. We will store the info about the user in the *userId* field of the request body:
```
const User = require('../models/user')
//...

notesRouter.post('/', async (request, response, next) => {
  const body = request.body

  const user = await User.findById(body.userId)
  const note = new Note({
    content: body.content,
    important: body.important === undefined ? false : body.important,
    date: new Date(),
    user: user._id  })

  const savedNote = await note.save()
  user.notes = user.notes.concat(savedNote._id)  await user.save()  
  response.json(savedNote)
})
```

This also changes the *user* object. The *id* of the note is stored in the *notes* field.

## Populate

We would like our API to work such that when an HTTP Get request is made to the */api/users* route, the user objects would also contain the contents of the user's notes, and not just their id. In a relational database, this functionality would be implemented with a *join query*.

Mongoose accomplishes the join by doing multiple queries, which is different from join queries in relational databases which are *transactional*, meaning that the state of the databse does not change during the time that the query is made. With join queries in Mongoose, nothing can guarantee that the state between the collections being joined is consistent, meaning that if we make a query that joins the user and notes collections, the state of the collections may change during the query.

The Mongoose join is done with the populate method. Let's update the route that returns all users:
```
usersRouter.get('/', async (request, response) => {
  const users = await User    
    .find({}).populate('notes')

  response.json(users)
})
```

The populate method is chained after the *find* method making the initial query. The parameter given to the populate method defines that the *ids* referencing *note* objects in the *notes* field of the *user* document will be replaced by the referenced *note* documents.

We can use the populate parameter for choosing the fields we want to include from the documents. The selection of fields is done with the Mongo syntax:
```
usersRouter.get('/', async (request, response) => {
  const users = await User
    .find({}).populate('notes', { content: 1, date: 1 })

  response.json(users)
});
```

It's important to understand that the database does not know that the ids stored in the *notes* field reference documents in the notes collection. The functionality of the *populate* method of Mongoose is based on the fact that we have defined "types" to the references in the Mongoose schema with the *ref* option.
