# Token authentication

Users must be able to log into our application, and when a user is logged in, their user information must automatically be attached to any new notes they create.

We will now implement support for token-based authentication to the backend.

The principles of token-based authentication are depicted in the following sequence diagram:
```mermaid
    sequenceDiagram
    Note left of user: user fills in login form with username and password
    user->>browser: login button pressed
    browser->>backend: HTTP POST /api/login { username, password }
    Note left of backend: backend generates TOKEN that identifies user
    backend->>browser: TOKEN returned as message body
    Note left of browser: browser saves TOKEN
    Note left of user: user creates a note
    user->>browser: create note button pressed
    browser->>backend: HTTP POST /api/notes { content } TOKEN in header
    Note left of backend: backend identifies user from the TOKEN
    backend->>browser: 201 created
```

To implement the functionality for logging in, install the jsonwebtoken library, which allows us to generate JSON web tokens:
```
npm install jsonwebtoken
```

The code for login functionality goes to the file *controllers/login.js*:
```
const jwt = require('jsonwebtoken')
const bcrypt = require('bcrypt')
const loginRouter = require('express').Router()
const User = require('../models/user')

loginRouter.post('/', async (request, response) => {
  const { username, password } = request.body

  const user = await User.findOne({ username })
  const passwordCorrect = user === null
    ? false
    : await bcrypt.compare(password, user.passwordHash)

  if (!(user && passwordCorrect)) {
    return response.status(401).json({
      error: 'invalid username or password'
    })
  }

  const userForToken = {
    username: user.username,
    id: user._id,
  }

  const token = jwt.sign(userForToken, process.env.SECRET)

  response
    .status(200)
    .send({ token, username: user.username, name: user.name })
})

module.exports = loginRouter
```

The code starts by searching for the user from the database by the *username* attached to the request. Next, it checks the *password*, also attached to the request. Because the passwords themselves are not saved to the database, but *hashes* calculated from the passwords, the `bcrypt.compare` method is used to check if the password is correct.

If the user is not found, or the password is incorrect, the request is responded to with the status code 401 unauthorized. The reason for the failure is in the response body.

If the password is correct, a token is created with the method `jwt.sign`. The token contains the username and the user id in a digitally signed form.

The token has been digitally signed using a string from the environment variable *SECRET* as the *secret*. The digital signature ensures that only parties who know the secret can generate a valid token. The value for the environment variable must be set in the *.env* file.

A successful request is responded to with the status code *200 OK*. The generated token and the username of the user are sent back in the response body.

## Limiting creating new notes to logged-in users

Let's change creating new notes so that it is only possible if the post request has a valid token attached. The note is then saved to the notes list of the user identified by the token.

There are several ways of sending the token from the browser to the server. We will use the Authorization header. The header also tells which authentication scheme is used. This can be necessary if the server offers multiple ways to authenticate. Identifying the scheme tells the server how the attached credentials should be interpreted.

The *Bearer* scheme is suitable for our needs. This means that if the token is the string *eyJhbGciOiJIUzI1NiIsInR5c2VybmFtZSI6Im1sdXVra2FpIiwiaW* for example, the Authorization header will have the value:
```
Bearer eyJhbGciOiJIUzI1NiIsInR5c2VybmFtZSI6Im1sdXVra2FpIiwiaW
```

Creating new notes changes like so:
```
const jwt = require('jsonwebtoken')
// ...
const getTokenFrom = request => {  
    const authorization = request.get('authorization')  
    if (authorization && authorization.toLowerCase().startsWith('bearer ')) {    
        return authorization.substring(7)  
    }  
    return null
}

notesRouter.post('/', async (request, response) => {
  const body = request.body
  const token = getTokenFrom(request)  
  const decodedToken = jwt.verify(token, process.env.SECRET)  
  if (!decodedToken.id) {    
    return response.status(401).json({ error: 'token missing or invalid' })  
  }  
  const user = await User.findById(decodedToken.id)

  const note = new Note({
    content: body.content,
    important: body.important === undefined ? false : body.important,
    date: new Date(),
    user: user._id
  })

  const savedNote = await note.save()
  user.notes = user.notes.concat(savedNote._id)
  await user.save()

  response.json(savedNote)
})
```

The helper function `getTokenFrom` isolates the token from the *authorization* header. The validity of the token is checked with `jwt.verify`. The method also decodes the token, or returns the Object which the token was based on. If there is no token passed, it will return the error "jwt must be provided".

The object decoded from the token contains the *username* and *id* fields, which tell the server who made the request.

If the object decoded from the token does not contain the user's identity, error status code 401 unauthorized is returned, and the reason for the failure is sent in the response body.

When the identity of the maker of the request is resolved, the execution continues as before.

Using Visual Studio Code REST client to connect looks like this:
```
post http://localhost:3001/api/notes
Content-Type: application/json
Authorization: bearer eyJhbGciOiJIUzI1NiIsInR5c2VybmFtZSI6Im1sdXVra2FpIiwiaW

{
    "content": "New note",
    "important": true
}
```

## Error handling

Token verification can also cause a *JsonWebTokenError*. There are many possible reasons for a decoding error. The token can be faulty, falsified, or expired. Let's extend our errorHandler middleware to ake into account the different decoding errors.
```
const unknownEndpoint = (request, response) => {
  response.status(404).send({ error: 'unknown endpoint' })
}

const errorHandler = (error, request, response, next) => {
  if (error.name === 'CastError') {
    return response.status(400).send({
      error: 'malformatted id'
    })
  } else if (error.name === 'ValidationError') {
    return response.status(400).json({
      error: error.message 
    })
  } else if (error.name === 'JsonWebTokenError') {    
    return response.status(401).json({      
        error: 'invalid token'     
    })  
  }

  logger.error(error.message)

  next(error)
}
```

## Problems of Token-based authentication

Token authentication is easy to implement, but contains one problem. Once the API user gets a token, the API has a blind trust in the token holder. What if the access rights of the token holder should be revoked.

One solution is to limit the validity period of a token:
```
loginRouter.post('/', async (request, response) => {
  const { username, password } = request.body

  const user = await User.findOne({ username })
  const passwordCorrect = user === null
    ? false
    : await bcrypt.compare(password, user.passwordHash)

  if (!(user && passwordCorrect)) {
    return response.status(401).json({
      error: 'invalid username or password'
    })
  }

  const userForToken = {
    username: user.username,
    id: user._id,
  }

  // token expires in 60*60 seconds, that is, in one hour
  const token = jwt.sign(    
    userForToken,     
    process.env.SECRET,    
    { expiresIn: 60*60 }  
  )

  response
    .status(200)
    .send({ token, username: user.username, name: user.name })
})
```

Once the token expires, the client app needs to get a new token. Usually, this happens by forcing the user to re-login to the app. We should add this case to the error handling middleware:
```
...
else if (error.name === 'TokenExpiredError') {    
    return response.status(401).json({      
        error: 'token expired'    
    })  
}
...
```

The shorter the expiration time, the more safe the solution is. On the other hand, a short expiration time forces the user to login more frequently.

The other solution is to save info about each token to the backend database and check for each API request if the access right corresponding to the token is still valid. This way, access rights can be revoked at any time. This is often called a *server-side session*.

The negative aspect of server-side sessions is the increased complexity in the backend, and also the effect on performance, since the token validity needs to be checkd for each API request to the database. A database access is considerably slower compared to checking the validity of the token itself. That is why it is common to save the session corresponding to a token to a *key-value database* such as Redis that is limited in functionality compared to MongoDB or a relational database, but extremely fast in some usage scenarios.

When server-side sessions are used, the token is often just a random string that does not include any information about the user as is often the case when jwt-tokens are used. For each API request, the server fetches the relevant information about the identity of the user from the database. It is also common that instead of using Authorization header, *cookies* are used as the mechanism for transferring the token between the client and server.
