# Component state and event handlers
We will start using a new example:
```
const Hello = (props) => {
  return (
    <div>
      <p>
        Hello {props.name}, you are {props.age} years old
      </p>
    </div>
  )
}

const App = () => {
  const name = 'Peter'
  const age = 10

  return (
    <div>
      <h1>Greetings</h1>
      <Hello name="Maya" age={26 + 10} />
      <Hello name={name} age={age} />
    </div>
  )
}
```

## Component helper functions
Here is an expansion of the Hello component with a *helper function*:
```
const Hello = (props) => {
  const bornYear = () => {    
    const yearNow = new Date().getFullYear()    
    return yearNow - props.age  
  }
  return (
    <div>
      <p>
        Hello {props.name}, you are {props.age} years old
      </p>
      <p>So you were probably born in {bornYear()}</p>   
    </div>
  )
}
```

The logic for guessing the year of birth is separated into a function of its own, which is called when the component is rendered.

The person's age does not have to be passed as a parameter to the function, since it can directly access all props that are passed to the component.

While in Java, defining a function inside another function is cumbersome and not common, in JavaScript, it is a commonly used technique.

## Destructuring
JavaScript ES6 added the ability to *destructure* values from objects and arrays upon assignment.

In previous code, we had to reference data from `props` as `props.name` and `props.age`.
Since `props` is an object, we can streamline our component by assigning the values of the properties directly into variables `name` and `age` which we can use in our code:
```
const Hello = (props) => {
    const name = props.name
    const age = props.age

    ...
}
```

Destructuring makes this assignment even easier, since we can use it to extract and gather values of an object's properties into separate variables:
```
const Hello = (props) => {
    const { name, age } = props

    ...
}
```

We can take destructuring a step further, by directly destructuring the props passed into the component into separate variables in the parameters of the function:
```
const Hello = ({ name, age }) => {
    ...
}
```

## Page re-rendering
Let's start with the following App.js:
```
const App = (props) => {
  const {counter} = props
  return (
    <div>{counter}</div>
  )
}

export default App
```
And index.js:
```
import React from 'react'
import ReactDOM from 'react-dom/client'

import App from './App'

let counter = 1

ReactDOM.createRoot(document.getElementById('root')).render(
  <App counter={counter} />
)
```

Even if we add `counter += 1`, the component won't re-render. One way to get it to re-render is by calling the `render` method a second time, like so:
```
let counter = 1

const refresh = () => {
  ReactDOM.createRoot(document.getElementById('root')).render(
    <App counter={counter} />
  )
}

refresh()
counter += 1
refresh()
counter += 1
refresh()
```

In the above example, the component renders 3 times, first with the value 1, then 2, then 3. However, the values 1 and 2 will be displayed for such a short time, they will not be visible to the user.

Another way would be using `setInterval` to re-render:
```
setInterval(() => {
  refresh()
  counter += 1
}, 1000)
```

Making repeated calls to the render method is not the recommended way to re-render components, and next, we will look at a better way of accomplishing this effect.

## Stateful component
All our components until now have not contained any state that could change during the lifecycle of the component.

We will now add state to the `App` component with the help of React's state hook.

index.js:
```
import React from 'react'
import ReactDOM from 'react-dom/client'

import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

App.js:
```
import { useState } from 'react'
const App = () => {
  const [ counter, setCounter ] = useState(0)
  setTimeout(   
    () => setCounter(counter + 1),
    1000  )
  return (
    <div>{counter}</div>
  )
}

export default App
```

The function call to `useState` adds *state* to the component and renders it initialized with the value of 0. The function returns an array that contains two items. `counter` and `setCounter` are assigned using destructuring, which we looked at earlier.

`counter` is assigned the initial value of *state*, which is zero. `setCounter` is assigned to a function that can modify the state.

When `setCounter` is called from the timer set in `setTimeout`, React re-renders the component, which means the function body of the component gets re-executed. The second time the component function is executed, it calls the useState function and returns the new value of the state, 1. Executing the function body again also makes a new function call to `setTimeout`.

Every time `setCounter` modifies the state, it causes the component to re-render. The value of the state will be incremented again after one second, which will continue for as long as the application is running.

## Event handling
Here is an example of using a evvent handler function for clicking a button:
```
const App = () => {
  const [ counter, setCounter ] = useState(0)

  const handleClick = () => {    
    console.log('clicked')  
  }

  return (
    <div>
      <div>{counter}</div>
      <button onClick={handleClick}>       
        plus     
      </button>    
    </div>
  )
}
```

We set the value of the button's onClick attribute to be a reference to the `handleClick` function. Every click of the button causes the `handleClick` function to be called.

You can also directly define an event handler function in the value assignment of the onClick attribute:
```
<button onClick={() => setCounter(counter + 1)}>
    ...
</button>
```

## Event handler is a function
If we try to define the event handler in a simpler form, it will not work, and throw an error reading "Too many re-renders":
```
<button onClick={setCounter(counter + 1)}> 
  plus
</button>
```

An event handler is supposed to be either a function or a function reference. The above is a *function call*, because it uses the parentheses. This would be ok in other situations, but here, it causes too many re-renders. When React renders the component the first time, it executes the `setCounter` function, which will re-render the component. This causes React to execute the `setCounter` function again, which re-renders again, which causes an infinite loop of re-rendering.

## Passing state to child components
It's recommended to write React components that are small and reusable across the application, and even across projects.

One best practice in React is to lift the state up in the component hierarchy, ie. lift the shared state up to the closest common ancestor of several components.

If we create a Display component:
```
const Display = (props) => {
  return (
    <div>{props.counter}</div>
  )
}
```

And a Button component:
```
const Button = (props) => {
  return (
    <button onClick={props.onClick}>
      {props.text}
    </button>
  )
}
```

The App component now looks like this:
```
const App = () => {
  const [ counter, setCounter ] = useState(0)

  const increaseByOne = () => setCounter(counter + 1)
  const decreaseByOne = () => setCounter(counter - 1)  
  const setToZero = () => setCounter(0)

  return (
    <div>
      <Display counter={counter}/>
      <Button        
        onClick={increaseByOne}        
        text='plus'      
      />      
      <Button        
        onClick={setToZero}        
        text='zero'      
      />           
      <Button        
        onClick={decreaseByOne}        
        text='minus'      
      />               
    </div>
  )
}
```

The event handler is passed to the `Button` component through the `onClick` prop.
