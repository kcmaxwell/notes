# Testing React apps

Tests will be implemented with the Jest testing library. Jest is configured by default to apps created with create-react-app.

In addition to Jest, we also need another testing library that will help us render components for testing purposes. The current best option for this is react-testing-library.

Let's install the library with the command:
```
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

We also install jest-dom, which provides Jest-related helper methods.

Let's first write tests for the component that is responsible for rendering a note:
```
const Note = ({ note, toggleImportance }) => {
  const label = note.important
    ? 'make not important'
    : 'make important'

  return (
    <li className='note'>      
      {note.content}
      <button onClick={toggleImportance}>{label}</button>
    </li>
  )
}
```

Notice that the *li* element has the CSS classname *note*, that can be used to access the component in our tests.

## Rendering the component for tests

We will write our test in the *src/components/Note.test.js* file, which is in the same directory as the component itself.

The first test verifies that the component renders the contents of the note:
```
import React from 'react'
import '@testing-library/jest-dom/extend-expect'
import { render, screen } from '@testing-library/react'
import Note from './Note'

test('renders content', () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  render(<Note note={note} />)

  const element = screen.getByText('Component testing is done with react-testing-library')
  expect(element).toBeDefined()
})
```

After the initial configuration, the test renders the component with the render function provided by the react-testing-library.

Normally, React components are rendered to the *DOM*. The render method we used renders the components in a format that is suitable for tests without rendering them to the DOM.

We can use the object screen to access the rendered component. We use screen's method getByText to search for an element that has the note content and ensure that it exists.

## Running tests

Create-react-app configures tests to be run in watch mode by default, which means that the `npm test` command will not exit once the tests have finished, and will instead wait for changes to be made to the code. Once new changes to the code are saved, the tests are executed automatically, after which Jest goes back to waiting for new changes to be made.

Note that the console may issue a warning if you have not installed Watchman. Watchman is an application developed by Facebook that watches for changes that are made to files.

## Test file location

In React, there are two different conventions for the test file's location. One, as seen above, is to create test files in the same directory as the component being tested.

The other convention is to store the test files in a separate `test` directory.

The reason we choose to follow the convention of keeping test files in the same directory as their component is because it is configured by default in applications created by create-react-app.

## Searching for content in a component

One way is to use `screen.getByText` as seen above.

We could also use CSS selectors to find rendered elements by using the method querySelector of the object container that is one of the fields returned by the render:
```
test('renders content', () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  const { container } = render(<Note note={note} />)

  const div = container.querySelector('.note')  
  expect(div).toHaveTextContent(    
    'Component testing is done with react-testing-library'  
  )
})
```

There are also other methods, eg. getByTestId, that look for elements based on id-attributes that are inserted in to the code specifically for testing purposes.

## Debugging tests

Object `screen` has method debug that can be used to print the HTML of a component to the terminal.

Using it with no parameter will print the entire body of the HTML, while specifying a component as a parameter will return only the HTML of that component:
```
screen.debug()
screen.debug(element)
```

## Clicking buttons in tests

We can install a library user-event that makes simulating user input a bit easier:
```
npm install --save-dev @testing-library/user-event
```

Here is an example test of testing the button that toggles importance of a note:
```
import React from 'react'
import '@testing-library/jest-dom/extend-expect'
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Note from './Note'

// ...

test('clicking the button calls event handler once', async () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  const mockHandler = jest.fn()

  render(
    <Note note={note} toggleImportance={mockHandler} />
  )

  const user = userEvent.setup()
  const button = screen.getByText('make not important')
  await user.click(button)

  expect(mockHandler.mock.calls).toHaveLength(1)
})
```

The event handler is a mock function defined with Jest. A session is started to interact with the rendered component by calling `userEvent.setup()`. The test finds the button *based on the text* from the rendered component, and clicks the element. Clicking happens with the `click` method of the user-event library. The expectation of the test verifies that the *mock function* has been called exactly once.

Mock objects and functions are commonly used stub components in testing that are used for replacing dependencies of the components being tested. Mocks make it possible to return hardcoded responses, and to verify the number of times the mock functions are called and with what parameters.

## Tests for the *Togglable* component

First, add the *togglableContent* CSS classname to the div that returns the child components:
```
const Togglable = forwardRef((props, ref) => {
  // ...

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>
          {props.buttonLabel}
        </button>
      </div>
      <div style={showWhenVisible} className="togglableContent">        
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
})
```

The tests are shown below:
```
import React from 'react'
import '@testing-library/jest-dom/extend-expect'
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Togglable from './Togglable'

describe('<Togglable />', () => {
  let container

  beforeEach(() => {
    container = render(
      <Togglable buttonLabel="show...">
        <div className="testDiv" >
          togglable content
        </div>
      </Togglable>
    ).container
  })

  test('renders its children', async () => {
    await screen.findAllByText('togglable content')
  })

  test('at start the children are not displayed', () => {
    const div = container.querySelector('.togglableContent')
    expect(div).toHaveStyle('display: none')
  })

  test('after clicking the button, children are displayed', async () => {
    const user = userEvent.setup()
    const button = screen.getByText('show...')
    await user.click(button)

    const div = container.querySelector('.togglableContent')
    expect(div).not.toHaveStyle('display: none')
  })
})
```

The `beforeEach` function gets called before each test, which renders the *Togglable* component and saves the field `container` of the return value.

The first test verifies that the *Togglable* component renders its child component.

The remaining tests use the toHaveStyle method to verify that the child component of the *Togglable* component is not visible initially, by checking that the style of the *div* element contains `{ display: 'none' }`. Another test verifies that when the button is pressed, the component is visible, meaning that the style for hiding the component is no longer assigned to the component.

## Testing the forms

We can also simulate text input with *userEvent*. Let's make a test for the *NoteForm* component. The code of the component is as follows:
```
import { useState } from 'react'

const NoteForm = ({ createNote }) => {
  const [newNote, setNewNote] = useState('')

  const handleChange = (event) => {
    setNewNote(event.target.value)
  }

  const addNote = (event) => {
    event.preventDefault()
    createNote({
      content: newNote,
      important: Math.random() > 0.5,
    })

    setNewNote('')
  }

  return (
    <div className="formDiv">
      <h2>Create a new note</h2>

      <form onSubmit={addNote}>
        <input
          value={newNote}
          onChange={handleChange}
        />
        <button type="submit">save</button>
      </form>
    </div>
  )
}

export default NoteForm
```

The form works by calling the `createNote` function it received as props. The test is as follows:
```
import React from 'react'
import { render, screen } from '@testing-library/react'
import '@testing-library/jest-dom/extend-expect'
import NoteForm from './NoteForm'
import userEvent from '@testing-library/user-event'

test('<NoteForm /> updates parent state and calls onSubmit', async () => {
  const createNote = jest.fn()
  const user = userEvent.setup()

  render(<NoteForm createNote={createNote} />)

  const input = screen.getByRole('textbox')
  const sendButton = screen.getByText('save')

  await user.type(input, 'testing a form...')
  await user.click(sendButton)

  expect(createNote.mock.calls).toHaveLength(1)
  expect(createNote.mock.calls[0][0].content).toBe('testing a form...')
})
```

Tests get access to the input field using the function getByRole.

The method type of the userEvent is used to write text to the input field.

The first test expectation ensures that submitting the form calls the `createNote` mocked method. The second expectation checks that the event handler is called with the right parameters - that a note with the correct content is created when the form is filled.

## About finding the elements

If we assume our form has two input fields, the approach our test above uses to find the input field would cause an error:
```
const input = screen.getByRole('textbox')
```

The error is *TestingLibraryElementError: Found multiple elements with the role "textbox".* The error message also suggests using *getAllByRole* instead. We could fix the test as follows:
```
const inputs = screen.getAllByRole('textbox')

await user.type(inputs[0], 'testing a form...')
```

Method *getAllByRole* now returns an array and the right input field is the first element of the array. This approach relies on knowing the order of the input fields.

Often, input fields have a *placeholder* text to imply to the user what kind of input is expected. If we add this to our input fields, we can use the method getByPlaceholderText:
```
test('<NoteForm /> updates parent state and calls onSubmit', () => {
  const createNote = jest.fn()

  render(<NoteForm createNote={createNote} />) 

  const input = screen.getByPlaceholderText('write here note content')  const sendButton = screen.getByText('save')

  userEvent.type(input, 'testing a form...')
  userEvent.click(sendButton)

  expect(createNote.mock.calls).toHaveLength(1)
  expect(createNote.mock.calls[0][0].content).toBe('testing a form...')
})
```

The most flexible way of finding elements in tests is the method *querySelector* of the `container` object, which is returned by `render`. Any CSS selector can be used with this method for searching elements in tests. We could add a unique `id` to the input field, and then find the input element as follows:
```
const { container } = render(<NoteForm createNote={createNote} />)

const input = container.querySelector('#note-input')
```

Note that the method `getByText` looks for an element that has the **same text** that it has as a parameter, nothing more. If we want to look for an element that *contains* the text, we could add an extra option:
```
const element = screen.getByText(
  'Does not work anymore :(', { exact: false }
)
```

We could also use the command `findByText`, which returns a promise:
```
const element = await screen.findByText('Does not work anymore :(')
```

Another ByText method, `queryByText` returns the element, but *does not cause an exception* if the element is not found. We could use this to ensure that something *is not rendered* to the component:
```
test('does not render this', () => {
  const note = {
    content: 'This is a reminder',
    important: true
  }

  render(<Note note={note} />)

  const element = screen.queryByText('do not want this thing to be rendered')
  expect(element).toBeNull()
})
```

## Test coverage

We can easily find the coverage of our tests by running the command:
```
CI=true npm test -- --coverage
```

An HTML report will be generated to the *coverage/lcov-report* directory. The report will tell us the lines of untested code in each component.

## Snapshot testing

Jest offers a different alternative to testing called snapshot testing. It does not require writing tests, only that the developer adopts it.

The fundamental principle is to compare the HTML code defined by the component after it has changed to the HTML code that existed before it was changed.

If the snapshot notices some change in the HTML defined by the component, then either it is new functionality, or a "bug" caused by accident. Snapshot tests notify the developer if the HTML code of the component changes. The developer has to tell Jest if the change was desired or undesired. If the change to the HTML code is unexpected, it strongly implies a bug, and the developer can become aware of these potential issues easily thanks to snapshot testing.
