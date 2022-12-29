# Forms

Here is a method for an event handler for the `onSubmit` attribute of a HTML form:
```
const addNote = (event) => {
  event.preventDefault()
  console.log('button clicked', event.target)
}
```
The `event` parameter is the event that triggers the call to the event handler function.

The event handler immediately calls the `event.preventDefault()` method, which prevents the default action of submitting a form, since the default action would cause the page to reload.

The `event.target` written to the console is the form that uses the event handler.

## Controlled component

One way to access the data in the form's *input* element is through *controlled components*.

Trying to set the value of an input field in a form to a piece of state like below will cause an error:
```
<form onSubmit={addNote}>
    <input value={newNote} />        
    <button type="submit">save</button>
</form>  
```
The placeholder text stored as the initial value in `newNote` will appear in the *input* element, but the input text can't be edited. Since we assigned a piece of the *App* component's state as the *value* attribute of the input element, the *App* component now controls the behavior of the input element.

To enable editing of the input element and remove the error, we have to register an *event handler* that synchronizes the changes made to the input with the component's state. This is an event handler for the *onChange* attribute:
```
<form onSubmit={addNote}>
    <input
        value={newNote}
        onChange={handleNoteChange}        
    />
    <button type="submit">save</button>
</form>
```

The event handler will be called every time *a change occurs in the input element*. The event handler function receives the event object as its `event` parameter:
```
const handleNoteChange = (event) => {    
    console.log(event.target.value)    
    setNewNote(event.target.value)  
}
```

The `target` property of the event object now corresponds to the controlled *input* element, and `event.target.value` refers to the input value of that element.

Note that we did not need to call the `event.preventDefault() method like with the *onSubmit* event handler. This is because there is no default action that occurs on an input change, unlike on a form submission.

## Filtering Displayed Elements

Here is an example of filtering important notes from display:
```
import { useState } from 'react'
import Note from './components/Note'

const App = (props) => {
  const [notes, setNotes] = useState(props.notes)
  const [newNote, setNewNote] = useState('') 
  const [showAll, setShowAll] = useState(true)

  // ...

  const notesToShow = showAll    
    ? notes    
    : notes.filter(note => note.important === true)

  return (
    <div>
      <h1>Notes</h1>
      <div>        
        <button onClick={() => setShowAll(!showAll)}>
          show {showAll ? 'important' : 'all' }
        </button>
      </div>
      <ul>
        {notesToShow.map(note =>
            <Note key={note.id} note={note} />
        )}
      </ul>
      // ...
    </div>
  )
}
```

If the value of `showAll` is false, the `notesToShow` variable will be assigned to a list that only contains notes that have the `important` property set to true. Filtering is done with the help of the array filter method.

The button's text is also conditionally rendered based on the value of `showAll`.

