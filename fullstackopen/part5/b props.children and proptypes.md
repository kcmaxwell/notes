# props.children and proptypes

## Displaying the login form only when appropriate
Instead of defining the login form inside of *app.js*, we will separate it into its own module. We can control the display of the form using the existing `loginForm` method:
```
const App = () => {
  const [loginVisible, setLoginVisible] = useState(false)
  // ...

  const loginForm = () => {
    const hideWhenVisible = { display: loginVisible ? 'none' : '' }
    const showWhenVisible = { display: loginVisible ? '' : 'none' }

    return (
      <div>
        <div style={hideWhenVisible}>
          <button onClick={() => setLoginVisible(true)}>log in</button>
        </div>
        <div style={showWhenVisible}>
          <LoginForm
            username={username}
            password={password}
            handleUsernameChange={({ target }) => setUsername(target.value)}
            handlePasswordChange={({ target }) => setPassword(target.value)}
            handleSubmit={handleLogin}
          />
          <button onClick={() => setLoginVisible(false)}>cancel</button>
        </div>
      </div>
    )
  }

  // ...
}
```

The *App* components state now contains the boolean *loginVisible*, which defines if the login form should be shown to the user or not.

The value of `loginVisible` is toggled with two buttons. The visibility of the component is defined by giving the component an inline style rule, where the value of the display property is *none* if we do not want the component to be displayed.

## The component's children, aka. props.children

The code related to managing the visibility of the login form could be considered to be its own logical entity, and for this reason, it would be good to extract it from the *App* component into a separate component.

Our goal is to implement a new *Togglable* component that can be used in the following way:
```
<Togglable buttonLabel='login'>
  <LoginForm
    username={username}
    password={password}
    handleUsernameChange={({ target }) => setUsername(target.value)}
    handlePasswordChange={({ target }) => setPassword(target.value)}
    handleSubmit={handleLogin}
  />
</Togglable>
```

The way that the component is used is slightly different from our previous components. The component has both opening and closing tags that surround a *LoginForm* component. In React terminology, *LoginForm* is a child component of *Togglable*.

The code for the *Togglable* component is shown below:
```
import { useState } from 'react'

const Togglable = (props) => {
  const [visible, setVisible] = useState(false)

  const hideWhenVisible = { display: visible ? 'none' : '' }
  const showWhenVisible = { display: visible ? '' : 'none' }

  const toggleVisibility = () => {
    setVisible(!visible)
  }

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>{props.buttonLabel}</button>
      </div>
      <div style={showWhenVisible}>
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
}

export default Togglable
```

The new part of the code is props.children, which is used for referencing the child components of the component. The child components are the React elements that we define between the opening and closing tags of a component.

Unlike the "normal" props we've seen before, *children* is automatically added by React and always exists. If a component is defined with an automatically closing tag (/>), then *props.children* is an empty array.

## References to components with ref

After a new note is created, it would make sense to hide the new note form. Currently, the form stays visible. There is a slight problem with hiding the form. The visibility is controlled with the *visible* variable inside of the *Togglable* component. How can we access it outside of the component?

We will use the ref mechanism of React, which offers a reference to the component. Let's make the following change to the *App* component:
```
import { useState, useEffect, useRef } from 'react'
const App = () => {
  // ...
  const noteFormRef = useRef()
  const noteForm = () => (
    <Togglable buttonLabel='new note' ref={noteFormRef}>      
        <NoteForm createNote={addNote} />
    </Togglable>
  )

  // ...
}
```

The useRef hook is used to create a *noteFormRef* ref that is assigned to the *Togglable* component containing the note creation form. The *noteFormRef* variable acts as a reference to the component. This hook ensures the same reference (ref) that is kept throughout re-renders of the component.

We also make the following changes to the *Togglable* component:
```
import { useState, forwardRef, useImperativeHandle } from 'react'
const Togglable = forwardRef((props, refs) => {  
  const [visible, setVisible] = useState(false)

  const hideWhenVisible = { display: visible ? 'none' : '' }
  const showWhenVisible = { display: visible ? '' : 'none' }

  const toggleVisibility = () => {
    setVisible(!visible)
  }

  useImperativeHandle(refs, () => {    
    return {      
        toggleVisibility    
    }  
  })

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>{props.buttonLabel}</button>
      </div>
      <div style={showWhenVisible}>
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
})
export default Togglable
```

The function that creates the component is wrapped inside of a *forwardRef* function call. This way, the component can access the ref that is assigned to it.

The component uses the *useImperativeHandle* hook to make its *toggleVisibility* function available outside of the component.

We can now hide the form by calling *noteFormRef.current.toggleVisibility()* after a new note has been created.

## One point about components

When we define a component in React and use it like this:
```
<div>
  <Togglable buttonLabel="1" ref={togglable1}>
    first
  </Togglable>

  <Togglable buttonLabel="2" ref={togglable2}>
    second
  </Togglable>

  <Togglable buttonLabel="3" ref={togglable3}>
    third
  </Togglable>
</div>
```

We create *three separate instances of the component* that all have their separate state.

The *ref* attribute is used for assigning a reference to each of the components in the variables *togglable1*, *togglable2*, and *togglable3*.

## PropTypes

The *Togglable* component assumes that it is given the text for the button via the *buttonLabel* prop. If we forget to define it to the component, the application works, but the browser renders a button that has no label text.

We would like to enforce that when the *Togglable* component is used, the button label text prop must be given a value.

The expected and required props of a component can be defined with the prop-types package. Let's install the package:
```
npm install prop-types
```

We can define the *buttonLabel* prop as a mandatory or *required* string-type prop as shown below:
```
import PropTypes from 'prop-types'

const Togglable = React.forwardRef((props, ref) => {
  // ..
})

Togglable.propTypes = {
  buttonLabel: PropTypes.string.isRequired
}
```

The console will display a "Failed prop type" error message if the prop is left undefined.

Here is a definition of PropTypes in the *LoginForm* component:
```
import PropTypes from 'prop-types'

const LoginForm = ({
   handleSubmit,
   handleUsernameChange,
   handlePasswordChange,
   username,
   password
  }) => {
    // ...
  }

LoginForm.propTypes = {
  handleSubmit: PropTypes.func.isRequired,
  handleUsernameChange: PropTypes.func.isRequired,
  handlePasswordChange: PropTypes.func.isRequired,
  username: PropTypes.string.isRequired,
  password: PropTypes.string.isRequired
}
```

If the type of a passed prop is wrong, this will display a warning.
