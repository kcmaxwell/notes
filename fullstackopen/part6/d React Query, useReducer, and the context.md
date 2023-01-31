# React Query, useReducer, and the context

We will look at a few more different ways to manage the state of an application.

Let's start a note application from scratch. The first version is as follows:
```
const App = () => {
  const addNote = async (event) => {
    event.preventDefault()
    const content = event.target.note.value
    event.target.note.value = ''
    console.log(content)
  }

  const toggleImportance = (note) => {
    console.log('toggle importance of', note.id)
  }

  const notes = []

  return(
    <div>
      <h2>Notes app</h2>
      <form onSubmit={addNote}>
        <input name="note" />
        <button type="submit">add</button>
      </form>
      {notes.map(note =>
        <li key={note.id} onClick={() => toggleImportance(note)}>
          {note.content} 
          <strong> {note.important ? 'important' : ''}</strong>
        </li>
      )}
    </div>
  )
}

export default App
```

## Managing data on the server with the React Query library

We shall use the React Query library to store and manage data retrieved from the server.

Install the library with the command:
```
npm install react-query
```

A few additions to the file *index.js* are needed to pass the library functions to the entire application:
```
import React from 'react'
import ReactDOM from 'react-dom/client'
import { QueryClient, QueryClientProvider } from 'react-query'
import App from './App'

const queryClient = new QueryClient()

ReactDOM.createRoot(document.getElementById('root')).render(
  <QueryClientProvider client={queryClient}>    
    <App />
  </QueryClientProvider>
)
```

We can now retrieve the notes in the *App* component. The code expands as follows:
```
import { useQuery } from 'react-query'import axios from 'axios'
const App = () => {
  // ...

  const result = useQuery(
    'notes',
    () => axios.get('http://localhost:3001/notes').then(res => res.data)
  )  console.log(result)

  if ( result.isLoading ) {
    return <div>loading data...</div>
  }

  const notes = result.data
  
  return (
    // ...
  )
}
```

Retrieving data from the server is still done in the familiar way with the Axios *get* method. However, the Axios method call is now wrapped in a *query* formed with the *useQuery* function. The first parameter of the function call is a string *notes* which acts as a key to the query defined, i.e. the list of notes.

The return value of the *useQuery* function is an object that indicates the status of the query.

The first time the component is rendered, the query is still in *loading* state, i.e. the associated HTTP request is pending. At this stage, only the following is rendered:
```
<div>loading data...</div>
```

However, the HTTP request is completed very quickly and this text will not be visible. When the request is completed, the component is rendered again. The query is in the state *success* on the second rendering, and the field *data* of the query object contains the data returned by the request, i.e. the list of notes that is rendered on the screen.

So the application retrieves data from the server and renders it on the screen without using the React hooks *useState* and *useEffect* at all. The data on the server is now entirely under the administration of the React Query library, and the application does not need the state defined with React's *useState* hook at all.

Let's move the function making the actual HTTP request to its own file *requests.js*:
```
import axios from 'axios'

export const getNotes = () =>
  axios.get('http://localhost:3001/notes').then(res => res.data)
```

The *App* component is now slightly simplified:
```
import { useQuery } from 'react-query' 
import { getAnecdotes } from './requests'
const App = () => {
  // ...

  const result = useQuery('notes', getNotes)
  // ...
}
```

## Synchronizing data to the server using React Query

Data is already successfully retrieved from the server. Next, we'll make sure that the added and modified data is stored on the server. Let's start by adding new notes.

Let's make a function *createNote* to the file *requests.js* for saving new notes:
```
import axios from 'axios'

const baseUrl = 'http://localhost:3001/notes'

export const getNotes = () =>
  axios.get(baseUrl).then(res => res.data)

export const createNote = newNote =>
  axios.post(baseUrl, newNote).then(res => res.data)
```

The *App* component will change as follows:
```
import { useQuery, useMutation } from 'react-query'
import { getNotes, createNote } from './requests'

const App = () => {
  const newNoteMutation = useMutation(createNote)

  const addNote = async (event) => {
    event.preventDefault()
    const content = event.target.note.value
    event.target.note.value = ''
    newNoteMutation.mutate({ content, important: true })
  }

  // ...
}
```

To create a new note, a mutation is defined using the function *useMutation*:
```
const newNoteMutation = useMutation(createNote)
```

The parameter is the function we added to the file *requests.js*, which uses Axios to send a new note to the server.

The event handler *addNote* performs the mutation by calling the mutation object's function *mutate* and passing the new note as a parameter:
```
newNoteMutation.mutate({ content, important: true })
```

Our solution does save a new note to the server, but it is not updated on the screen. In order to render a new note as well, we need to tell React Query that the old result of the query whose key is the string *notes* should be *invalidated*. This can be done by defining the appropriate *onSuccess* callback function to the mutation:
```
import { useQuery, useMutation, useQueryClient } from 'react-query'
import { getNotes, createNote } from './requests'

const App = () => {
  const queryClient = useQueryClient()

  const newNoteMutation = useMutation(createNote, {
    onSuccess: () => {
      queryClient.invalidateQueries('notes')
    },
  })

  // ...
}
```

Now that the mutation has been successfully executed, a function call is made to:
```
queryClient.invalidateQueries('notes')
```

This in turn causes React Query to automatically update a query with the key *notes*, i.e. fetch the notes from the server. As a result, the application renders the up-to-date state on the server, so the added note will also be rendered.

Let's also implement the change in the importance of notes. A function for updating notes is added to the file *requests.js*:
```
export const updateNote = updatedNote =>
  axios.put(`${baseUrl}/${updatedNote.id}`, updatedNote).then(res => res.data)
```

Updating the note is also done by mutation. The *App* component expands as follows:
```
import { useQuery, useMutation, useQueryClient } from 'react-query' 
import { getNotes, createNote, updateNote } from './requests'

const App = () => {
  // ...

  const updateNoteMutation = useMutation(updateNote, {
    onSuccess: () => {
      queryClient.invalidateQueries('notes')
    },
  })

  const toggleImportance = (note) => {
    updateNoteMutation.mutate({...note, important: !note.important })
  }

  // ...
}
```

Again, a mutation was created that invalidated the query *notes* so that the updated note is rendered correctly. Using mutation is easy, the method *mutate* receives a note as a parameter, the importance of which has been changed to the negation of the old value.

## Optimizing the performance

The application works well, and the code is relatively simple. The ease of making changes to the list of notes is quite surprising. For example, when we change the importance of a note, invalidating the query *notes* is enough for the application data to be updated:
```
  const updateNoteMutation = useMutation(updateNote, {
    onSuccess: () => {
      queryClient.invalidateQueries('notes')
    },
  })
```

The consequence of this is that after the PUT request that causes the note change, the application makes a new GET request to retrieve the query data from the server.

If the amount of data retrieved by the application is not large, it doesn't really matter. After all, from a browser-side functionality point of view, an extra HTTP GET request doesn't really matter, but in some situations, it might put a strain on the server.

If necessary, it is also possible to optimize performance by manually updating the query state maintained by React Query.

The change for the mutation adding a new note is as follows:
```
const App = () => {
  const queryClient =  useQueryClient() 

  const newNoteMutation = useMutation(createNote, {
    onSuccess: (newNote) => {
      const notes = queryClient.getQueryData('notes')
      queryClient.setQueryData('notes', notes.concat(newNote))
    }
  })
  // ...
}
```

That is, in the *onSuccess* callback, the *queryClient* object first reads the existing *notes* state of the query and updates it by adding a new note, which is obtained as a parameter of the callback function. The value of the parameter is the value returned by the function *createNote*, defined in the file *requests.js* as follows:
```
export const createNote = newNote =>
  axios.post(baseUrl, newNote).then(res => res.data)
```

If we closely follow the browser's network tab, we notice that React Query retrieves all notes as soon as we move the cursor to the input field. By reading the documentation, we notice that the default functionality of React Query's queries is that the queries (whose status is *stale*) are updated when *window focus*, i.e. the active element of the application's user interface, changes. If we want, we can turn off the functionality by creating a query as follows:
```
const App = () => {
  // ...
  const result = useQuery('notes', getNotes, {
    refetchOnWindowFocus: false
  })

  // ...
}
```

You can see how often React Query causes the application to be re-rendered using console.log. The rule of thumb for re-rendering is that it happens at least whenever there is a need for it, i.e. when the state of the query changes.

React Query is a versatile library that, based on what we have already seen, simplifies the application. However, it does not make more complex state management solutions like Redux unnecessary. React Query can partially replace the state of the application in some cases, but as the documentation states:
- React Query is a *server-state library*, responsible for managing asynchronous operations between your server and client
- Redux, etc are *client-state libraries* that can be used to store asynchronous data, albeit inefficiently when compared to a tool like React Query

So React Query is a library that maintains the *server state* in the frontend, i.e. acts as a cache for what is stored on the server. React Query simplifies the processing of data on the server, and in some cases, can eliminate the need for data on the server to be saved in the frontend state.

Most React applications need not only have a way to temporarily store the served data, but also some solution for how the rest of the frontend state (e.g. the state of forms or notifications) is handled.
