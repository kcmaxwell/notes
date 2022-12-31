# Altering data in server

## REST

In REST terminology, we refer to individual data objects, such as the notes in our application, as *resources*. Every resource has a unique address associated with it - its URL.

Resources are fetched from the server with HTTP GET requests. Creating a new resource for storing a note is done by making an HTTP POST request to the *notes* URL, according to the REST convention json-server adheres to. The data for the new resource is sent in the *body* of the request.

json-server requires all data to be sent in JSON format. This means that the data must be a correctly formatted string, and the request must contain the *Content-Type* request header with the value *application/json*.

## Sending data to the server

Here is an example event handler to add a note to the server by using a POST with axios:
```
addNote = event => {
  event.preventDefault()
  const noteObject = {
    content: newNote,
    date: new Date(),
    important: Math.random() > 0.5, // randomized importance field
  }

  axios
    .post('http://localhost:3001/notes', noteObject)
    .then(response => {
      setNotes(notes.concat(response.data))     
      setNewNote('')    
    })
}
```

We create a new object for the note, but omit the *id* property, since it's better to let the server generate ids for our resources.

Since the data we sent in the POST request was a JavaScript object, axios automatically knew to set the appropriate *application/json* value for the *Content-Type* header.

The new note returned by the backend server is added to the list of notes in our application's state by using the `setNotes` function, then resetting the note creation form.

## Editing existing data on the server

Here is an example event handler for toggling the importance of a given existing note:
```
const toggleImportanceOf = id => {
  const url = `http://localhost:3001/notes/${id}`
  const note = notes.find(n => n.id === id)
  const changedNote = { ...note, important: !note.important }

  axios.put(url, changedNote).then(response => {
    setNotes(notes.map(n => n.id !== id ? n : response.data))
  })
}
```

First, we define a url for the existing note resource based on its id. We find the existing note on the local browser side and create a *new object* that is a copy of the old note, except for its important property, which we toggle. The new note is then sent with a PUT request to the backend, where it will replace the old object. Finally, we use the map method to create a new array where all notes are the same except the one we changed, which will be replaced with `response.data`, the new note.

Note that we must create a new object for `changedNote` rather than editing it directly, because it is a part of state, and state must never be mutated directly in React.

## Extracting Backend Communication into a Separate Module

Here is a separate module that contains functions that communicate with the backend. Note that these functions still return a promise, as the `then` method of a promise also returns a promise. You can chain another `then` onto these function calls and access the `response.data` object directly.
```
import axios from 'axios'
const baseUrl = 'http://localhost:3001/notes'

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

const create = newObject => {
  const request = axios.post(baseUrl, newObject)
  return request.then(response => response.data)
}

const update = (id, newObject) => {
  const request = axios.put(`${baseUrl}/${id}`, newObject)
  return request.then(response => response.data)
}

export default { 
  getAll: getAll, 
  create: create, 
  update: update 
}
```

For more reading on Promises:
https://javascript.info/promise-chaining
https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/async%20%26%20performance/ch3.md

## Cleaner Syntax for Defining Object Literals

In the above code, we expored an object like this:
```
export default { 
  getAll: getAll, 
  create: create, 
  update: update 
}
```

The labels to the left of the colon in the object definition are the *keys* of the object, whereas the ones to the right of it are *variables* that are defined inside the module.

Since the names of the keys and the assigned variables are the same, we can write the object definition with a more compact syntax:
```
export default { getAll, create, update }
```

This is a shorthand property definition syntax added in ES6 JavaScript.

## Promises and Errors

When an HTTP request fails, the associated promise is *rejected*. The rejection of a promise is handled by providing the `then` method with a second callback function, which is called in the situation where the promise is rejected.

The more common way of adding a handler for rejected promises is to use the `catch` method. For example:
```
axios
  .get('http://example.com/probably_will_fail')
  .then(response => {
    console.log('success!')
  })
  .catch(error => {
    console.log('fail')
  })
```

If the request fails, the event handler registered with the `catch` method gets called.

You can place the `catch` method at the end of a promise chain, and it will be called once any promise in the chain throws an error and the promise becomes rejected.
