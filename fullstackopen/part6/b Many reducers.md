# Many reducers

Let's continue our work with the simplified redux version of our notes application.

To ease our development, let's change our reducer so that the store gets initialized with a state that contains a couple of notes:
```
const initialState = [
  {
    content: 'reducer defines how redux store works',
    important: true,
    id: 1,
  },
  {
    content: 'state of store can contain any data',
    important: false,
    id: 2,
  },
]

const noteReducer = (state = initialState, action) => {
  // ...
}

// ...
export default noteReducer
```

## Store with complex state

Let's implement filtering for the notes that are displayed to the user. The user interface for the filters will be implemented with radio buttons.

Let's start with a very simple and straightforward implementation:
```
import NewNote from './components/NewNote'
import Notes from './components/Notes'

const App = () => {
  const filterSelected = (value) => {    console.log(value)  }
  return (
    <div>
      <NewNote />
      <div>        
        all          <input type="radio" name="filter"          
          onChange={() => filterSelected('ALL')} />        
        important    <input type="radio" name="filter"          
          onChange={() => filterSelected('IMPORTANT')} />        
        nonimportant <input type="radio" name="filter"          
          onChange={() => filterSelected('NONIMPORTANT')} />      
      </div>      
      <Notes />
    </div>
  )
}
```

Since the *name* attribute of all the radio buttons is the same, they form a *button group* where only one option can be selected.

The buttons have a change handler that currently only prints the string associated with the clicked button to the console.

We decide to implement the filter functionality by storing *the value of the filter* in the redux store in addition to the notes themselves. The state of the store should look like this after making these changes:
```
{
  notes: [
    { content: 'reducer defines how redux store works', important: true, id: 1},
    { content: 'state of store can contain any data', important: false, id: 2}
  ],
  filter: 'IMPORTANT'
}
```

Only the array of notes is stored in the state of our current implementation. In the new implementation, the state object has two properties, *notes* that contains the array of notes, and *filter* that contains a string indicating which notes should be displayed to the user.

## Combined reducers

We could modify our current reducer to deal with the new shape of the state. However, a better solution in this situation is to define a new separate reducer for the state of the filter:
```
const filterReducer = (state = 'ALL', action) => {
  switch (action.type) {
    case 'SET_FILTER':
      return action.filter
    default:
      return state
  }
}
```

The actions for changing the state of the filter look like this:
```
{
  type: 'SET_FILTER',
  filter: 'IMPORTANT'
}
```

Let's also create a new `action creator` function. We will write the code for the action creator in a new *src/reducers/filterReducer.js* module:
```
const filterReducer = (state = 'ALL', action) => {
  // ...
}

export const filterChange = filter => {
  return {
    type: 'SET_FILTER',
    filter,
  }
}

export default filterReducer
```

We can create the actual reducer for our application by combining the two existing reducers with the *combineReducers* function.

Let's define the combined reducer in the *index.js* file:
```
import React from 'react'
import ReactDOM from 'react-dom/client'
import { createStore, combineReducers } from 'redux'import { Provider } from 'react-redux' 
import App from './App'

import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const reducer = combineReducers({  
    notes: noteReducer,  
    filter: filterReducer
})

const store = createStore(reducer)

console.log(store.getState())

ReactDOM.createRoot(document.getElementById('root')).render(
  /*
  <Provider store={store}>
    <App />
  </Provider>,
  */
  <div />
)
```

We comment out the *App* component as we have not refactored it yet.

The state of the sotre defined by the reducer above is an object with two properties: *notes* and *filter*. The value of the *notes* property is defined by the *noteReducer*, which does not have to deal with the other properties of the state. Likewise, the *filter* property is managed by the *filterReducer*.

Note that if we add console log statements at the beginning of both reducers, we might think that every action is duplicated, because there will be print statements from both reducers after each action. There is no bug, though, as the combined reducer works in such a way that *every action* gets handled in *every part* of the combined reducer. Typically, only one reducer is interested in any given action, but there are situations where multiple reducers change their respective parts of the state based on the same action.

## Redux Toolkit

As we have seen so far, Redux's configuration and state management implementation requires a lot of effort. This is manifested, for example, in the reducer and action creator-related code, which has somewhat repetitive boilerplate code. *Redux Toolkit* is a library that solves these common Redux-related problems. The library greatly simplifies the configuration of the Redux store, and offers a large variety of tools to ease state management.

First, we need to install the library:
```
npm install @reduxjs/toolkit
```

Next, open the *index.js* file which currently creates the Redux store. Instead of Redux's `createStore` function, let's create the store using Redux Toolkit's `configureStore` function:
```
import React from 'react'
import ReactDOM from 'react-dom/client'
import { Provider } from 'react-redux'
import { configureStore } from '@reduxjs/toolkit'import App from './App'

import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const store = configureStore({  
    reducer: {    
        notes: noteReducer,    
        filter: filterReducer  
    }
})

console.log(store.getState())

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <App />
  </Provider>
)
```

We already removed some code since we do not need the `combineReducers` function. We will soon see that the `configureStore` function has many additional benefits.

Let's move on to refactoring the reducers, which brings forth the benefits of the Redux Toolkit. With Redux Toolkit, we can easily create reducer and related action creators using the `createSlice` function. We can use `createSlice` to refactor the reducer and action creators in the *reducers/noteReducer.js* file:
```
import { createSlice } from '@reduxjs/toolkit'
const initialState = [
  {
    content: 'reducer defines how redux store works',
    important: true,
    id: 1,
  },
  {
    content: 'state of store can contain any data',
    important: false,
    id: 2,
  },
]

const generateId = () =>
  Number((Math.random() * 1000000).toFixed(0))

const noteSlice = createSlice({  
    name: 'notes',  
    initialState,  
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
        }  
    },
})
```

The `createSlice` function's `name` parameter defines the prefix which is used in the action's type values. For example, the `createNote` action defined later will have the type value of `notes/createNote`. It is a good practice to give the parameter a value, which is unique among the reducers. This way, there won't be unexpected collisions between the application's action type values. The `initialState` parameter defines the reducer's initial state. The `reducers` parameter takes the reducer itself as an object, of which functions handle state changes cause by certain actions. Note that the `action.payload` in the function contains the argument provided by calling the action creator.

For example:
```
dispatch(createNote('Redux Toolkit is awesome!'))
```

This dispatch call corresponds to dispatching the following object:
```
dispatch({ type: 'notes/createNote', payload: 'Redux Toolkit is awesome!' })
```

You might notice that we used the `state.push` function, which seems to violate the reducer's immutability principle from before.

Redux Toolkit utilizes the Inmer library with reducers created by the `createSlice` function, which makes it possible to mutate the `state` argument inside the reducer. Inmer uses the mutated state to produce a new, immutable state, and thus, the state changes remain immutable. Note that `state` can be changed without mutating it, as we have done before. In this case, the function *returns* the new state. Nevertheless, mutating the state will often come in handy, especially when a complex state needs to be updated.

The `createSlice` function returns an object containing the reducer as well as the action creators defined by the `reducers` parameter. The reducer can be accessed by the `noteSlice.reducer` property, whereas the action creators are accessed by the `noteSlice.actions` property. We can produce the file's exports like so:
```
const noteSlice = createSlice(/* ... */)

export const { createNote, toggleImportanceOf } = noteSlice.actions
export default noteSlice.reducer
```

The imports in other files will work just as they did before:
```
import noteReducer, { createNote, toggleImportanceOf } from './reducers/noteReducer'
```

## Redux DevTools

Redux DevTools is a Chrome addon that offers useful development tools for Redux. It can be used, for example, to inspect the Redux store's state and dispatch actions through the browser's console. When the store is created using Redux Toolkit's `configureStore` function, no additional configuration is needed for Redux DevTools to work.
