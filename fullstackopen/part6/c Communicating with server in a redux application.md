# Communicating with server in a redux application

Let's expand the application so that notes are stored in the backend. We'll use json-server, familiar from part 2.

The initial state of the database is stored in the file *db.json*, which is placed in the root of the project:
```
{
  "notes": [
    {
      "content": "the app state is in redux store",
      "important": true,
      "id": 1
    },
    {
      "content": "state changes are made with actions",
      "important": false,
      "id": 2
    }
  ]
}
```

We'll install json-server for the project, and add the following line to the *scripts* part of the file *package.json*:
```
"scripts": {
  "server": "json-server -p3001 --watch db.json",
  // ...
}
```

Next, we'll create a method in the file *services/notes.js*, which uses *axios* to fetch data from the backend:
```
import axios from 'axios'

const baseUrl = 'http://localhost:3001/notes'

const getAll = async () => {
  const response = await axios.get(baseUrl)
  return response.data
}

export default { getAll }
```

We'll change the initialization of the state in *noteReducer*, so that by default, there are no notes:
```
const noteSlice = createSlice({
  name: 'notes',
  initialState: [],  
  // ...
})
```

Let's also add a new action, `appendNote`, for adding a note object:
```
const noteSlice = createSlice({
  name: 'notes',
  initialState: [],
  reducers: {
    createNote(state, action) {
      const content = action.payload

      state.push({
        content,
        important: false,
        id: generateId(),
      })
    },
    toggleImportanceOf(state, action) {
      const id = action.payload

      const noteToChange = state.find(n => n.id === id)

      const changedNote = { 
        ...noteToChange, 
        important: !noteToChange.important 
      }

      return state.map(note =>
        note.id !== id ? note : changedNote 
      )     
    },
    appendNote(state, action) {      
        state.push(action.payload)    
    }  
  },
})

export const { createNote, toggleImportanceOf, appendNote } = noteSlice.actions
export default noteSlice.reducer
```

A quick way to initialize the notes state based on the data received from the server is to fetch the notes in the *index.js* file and dispatch an action using the `appendNote` action creator for each individual note:
```
// ...
import noteService from './services/notes'import noteReducer, { appendNote } from './reducers/noteReducer'
const store = configureStore({
  reducer: {
    notes: noteReducer,
    filter: filterReducer,
  }
})

noteService.getAll().then(notes =>  
notes.forEach(note => {    
    store.dispatch(appendNote(note))  
  })
)

// ...
```

Instead of dispatching multiple actions to fill the notes array, let's add an action creator `setNotes` which can be used to directly replace the notes array:
```
const noteSlice = createSlice({
  name: 'notes',
  initialState: [],
  reducers: {
    createNote(state, action) {
      const content = action.payload

      state.push({
        content,
        important: false,
        id: generateId(),
      })
    },
    toggleImportanceOf(state, action) {
      const id = action.payload

      const noteToChange = state.find(n => n.id === id)

      const changedNote = { 
        ...noteToChange, 
        important: !noteToChange.important 
      }

      return state.map(note =>
        note.id !== id ? note : changedNote 
      )     
    },
    appendNote(state, action) {
      state.push(action.payload)
    },
    setNotes(state, action) {      
        return action.payload    
    }  
  },
})

export const { createNote, toggleImportanceOf, appendNote, setNotes } = noteSlice.actions
export default noteSlice.reducer
```

Now, our code in *index.js* looks cleaner:
```
// ...
import noteService from './services/notes'
import noteReducer, { setNotes } from './reducers/noteReducer'

const store = configureStore({
  reducer: {
    notes: noteReducer,
    filter: filterReducer,
  }
})

noteService.getAll().then(notes =>
  store.dispatch(setNotes(notes))
)
```

If we want to move this initialization of the notes array to the *App* component, we can put it into a `useEffect` hook so it will load on initialization of our app:
```
const App = () => {
  const dispatch = useDispatch()
  useEffect(() => {
    noteService
      .getAll().then(notes => dispatch(setNotes(notes)))
  }, [dispatch])
  // ...
}
```

We also need to add `dispatch` to the dependency array.

Let's also add a service to create a new note in *services/notes.js*:
```
const createNew = async (content) => {  
    const object = { content, important: false }  
    const response = await axios.post(baseUrl, object)  
    return response.data
}
```

Because the backend generates ids for the notes, we'll change the action creator `createNote` accordingly:
```
createNote(state, action) {
  state.push(action.payload)
}
```

## Asynchronous actions and Redux thunk

Our approach is quite good, but it is not great that the communication with the server happens inside the functions of the components. It would be better if the communication could be abstracted away from the components so that they don't have to do anything else but call the appropriate *action creator*. As an example, *App* would initialize the state of the application as follows:
```
const App = () => {
  const dispatch = useDispatch()

  useEffect(() => {
    dispatch(initializeNotes()))  
  }, [dispatch]) 

  // ...
}
```

And *NewNote* would create a new note as follows:
```
const NewNote = () => {
  const dispatch = useDispatch()
  
  const addNote = async (event) => {
    event.preventDefault()
    const content = event.target.note.value
    event.target.note.value = ''
    dispatch(createNote(content))
  }

  // ...
}
```

In this implementation, both components would dispatch an action without the need to know about the communication between the server that happens behind the scenes.

These kinds of *async actions* can be implemented with the *Redux Thunk* library. The use of the library doesn't need any additional configuration when the Redux store is created using the Redux Toolkit's `configureStore` function.

We can install it with the following command:
```
npm install redux-thunk
```

With Redux Thunk, it is possible to implement *action creators* which return a function instead of an object. The function receives Redux store's `dispatch` and `getState` methods as parameters. This allows, for example, implementation of asynchronous action creators, which first wait for the completion of a certain asynchronous operation, and after that, dispatch some action which changes the store's state.

We can define an action creator `initializeNotes` which initializes the notes based on the data received from the server:
```
// ...
import noteService from '../services/notes'
const noteSlice = createSlice(/* ... */)

export const { createNote, toggleImportanceOf, setNotes, appendNote } = noteSlice.actions

export const initializeNotes = () => {  
    return async dispatch => {    
        const notes = await noteService.getAll()    
        dispatch(setNotes(notes))  
    }
}
export default noteSlice.reducer
```

In the inner function, the *asynchronous action*, the operation first fetches all the notes from the server and then *dispatches* the `setNotes` action, which adds them to the store.

The component *App* can now be defined as follows:
```
// ...
import { initializeNotes } from './reducers/noteReducer'
const App = () => {
  const dispatch = useDispatch()

  useEffect(() => {    
    dispatch(initializeNotes())   
  }, [dispatch]) 

  return (
    <div>
      <NewNote />
      <VisibilityFilter />
      <Notes />
    </div>
  )
}
```

Next, let's replace the createNote action creator created by the `createSlice` function with an asynchronous action creator:
```
// ...
import noteService from '../services/notes'

const noteSlice = createSlice({
  // ...
})

export const { toggleImportanceOf, appendNote, setNotes } = noteSlice.actions
export const initializeNotes = () => {
  // ...
}

export const createNote = content => {  
    return async dispatch => {    
        const newNote = await noteService.createNew(content)    
        dispatch(appendNote(newNote))  
    }

}
export default noteSlice.reducer
```

Let's also clean up *index.js* by moving the code for creating the Redux store into its own, *store.js* file:
```
import { configureStore } from '@reduxjs/toolkit'

import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const store = configureStore({
  reducer: {
    notes: noteReducer,
    filter: filterReducer
  }
})

export default store
```
