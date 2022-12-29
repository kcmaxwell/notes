## Complex state
If we need more complex state, we can use the `useState` function multiple times to create separate pieces of state.

The component's state or a piece of its state can be of any type. We can use an object, for example:
```
const App = () => {
  const [clicks, setClicks] = useState({
    left: 0, right: 0
  })

  const handleLeftClick = () => {
    const newClicks = { 
      left: clicks.left + 1, 
      right: clicks.right 
    }
    setClicks(newClicks)
  }

  const handleRightClick = () => {
    const newClicks = { 
      left: clicks.left, 
      right: clicks.right + 1 
    }
    setClicks(newClicks)
  }

  return (...)
}
```

We can define the new state object a bit more neatly using *object spread* syntax:
```
const handleLeftClick = () => {
  const newClicks = { 
    ...clicks, 
    left: clicks.left + 1 
  }
  setClicks(newClicks)
}
```

`{ ...clicks }` creates a new object that has copies of all of the properties in the `clicks` object. When we specify a particular property, eg. `left` in `{ ...clicks, left: clicks.left + 1 }`, the value of `left` in the new object will be `clicks.left + 1`.

The form of the above function can be simplified further:
```
const handleLeftClick = () =>
  setClicks({ ...clicks, left: clicks.left + 1 })
```

The example below is **not allowed**:
```
const handleLeftClick = () => {
  clicks.left++
  setClicks(clicks)
}
```
*It is forbidden in React to mutate state directly*, since it can result in unexpected side effects. Changing state has to be done by setting the state to a new object.

According to the React docs, it is recommended to split state into multiple state variables based on which values tend to change together. If they change independent of each other, keep them separated.

## Handling arrays
To initialize an array as a piece of state:
```
const [allClicks, setAll] = useState([])
```

And here is an example of how to change an array state:
```
const handleLeftClick = () => {
  setAll(allClicks.concat('L'))
  setLeft(left + 1)
}
```

The piece of state stored in `allClicks` is now set to be an array that contains all items of the previous state array plus the letter *L*. Adding the new item to the array is done using the concat method, which does not mutate the existing array, but returns a new copy of the array with the item added to it.

**Do not** use push on state arrays. It mutates the array directly, which will lead to bugs.

## Conditional rendering
Here is a new *History* component, which renders the clicking history:
```
const History = (props) => {  
    if (props.allClicks.length === 0) {
        return (      
            <div>        
                the app is used by pressing the buttons      
            </div>    
        )  
    }  
    return (    
        <div>      
            button press history: {props.allClicks.join(' ')}
        </div>  
    )
}

const App = () => {
  // ...

  return (
    <div>
      {left}
      <button onClick={handleLeftClick}>left</button>
      <button onClick={handleRightClick}>right</button>
      {right}
      <History allClicks={allClicks} />    
    </div>
  )
}
```

If no buttons have been clicked, the `allClicks` array is empty, and the component renders a div element with instructions. Otherwise, the component renders the clicking history.

The *History* component renders completely different React elements depending on the state of the application. This is called *conditional rendering*. There are also other ways of doing conditional rendering, which we will look at later.

## Old React
Currently, we are using the state hook to add state to our React components. This is a part of newer versions of React, and is available from version 16.8.0 onwards. Before the addition of hooks, there was no way to add state to functional components. Components requiring state had to be defined as class components, using the JavaScript class syntax.

## Debugging React applications

### The first rule of web development
**Keep the browser's developer console open at all times.**

When your code fails to compile, stop writing code and find and fix the problem **immediately**.

You can use basic print-based debugging using console.log. When doing this, use commas to separate items rather than the plus operator:
```
console.log('props value is', props)
```

You can pause the execution of your application code in the Chrome developer console's debugger by writing the command *debugger* anywhere in your code. Execution will pause once it executes the command, and you can go to the console tab to inspect the current state of variables.

You can also access the debugger by adding breakpoints in the *Sources* tab. You can execute code line by line on the right-hand side of the *Sources* tab. Inspecting the values of the component's variables can be done in the `Scope` section.

You can also add the React developer tools extension to Chrome. It adds a new Components tab to the developer tools, which can be used to inspect different React elements in the application, along with their state and props.

## Rules of Hooks
The `useState` function (as well as the `useEffect` function introduced later on) *must not be called* from inside of a loop, a conditional expression, or any place that is not a function defining a component. This must be done to ensure hooks are always called in the same order, and if this isn't the case, the application will behave erratically.

## Function that returns a function
Another way to define an event handler is to use a function that returns a function:
```
const App = () => {
  const [value, setValue] = useState(10)

  const hello = () => {    
    const handler = () => console.log('hello world')    
    return handler  
  }

  return (
    <div>
      {value}
      <button onClick={hello()}>button</button>
    </div>
  )
}
```

In the above code, the `onClick` event handler is set to the function call `hello()`. There is no problem because the function `hello` is returning a function defined inside of it, so the event handler still ends up being set to a function. 

It is equivalent to `onClick={() => console.log('hello world')}`.

Here is an example that shows a possible useful use case for this concept:
```
const App = () => {
  const [value, setValue] = useState(10)

  const hello = (who) => {    
    const handler = () => {      
        console.log('hello', who)    
    }    
    return handler  
  }

  return (
    <div>
      {value}
      <button onClick={hello('world')}>button</button>      
      <button onClick={hello('react')}>button</button>      
      <button onClick={hello('function')}>button</button>    
    </div>
  )
}
```

In the above example, each button gets its own individualized event handler by being able to pass arguments to the function returning the event handler function.

A less verbose version of the `hello` function above would be:
```
const hello = (who) => () => {
  console.log('hello', who)
}
```

We can also use the same trick to define event handlers that set the state of the component to a given value:
```
const App = () => {
  const [value, setValue] = useState(10)
  
  const setToValue = (newValue) => () => {    
    console.log('value now', newValue)  // print the new value to console    
    setValue(newValue)  
  }  

  return (
    <div>
      {value}
      <button onClick={setToValue(1000)}>thousand</button>      
      <button onClick={setToValue(0)}>reset</button>      
      <button onClick={setToValue(value + 1)}>increment</button>    
    </div>
  )
}
```

Choosing between the two presented ways of defining event handlers is mostly a matter of taste.

## Passing Event Handlers to Child Components
Here is an example of how to pass event handlers to a child component:
```
const Button = (props) => (
  <button onClick={props.handleClick}>
    {props.text}
  </button>
)

const App = () => {
    ...

    return (
        <div>
            ...
            <Button handleClick={() => setToValue(1000)} text="thousand" />
        </div>
    )
}
```

You only need to make sure you use the correct attribute names when passing props to the component.

## Do Not Define Components Within Components
**Never define components inside of other components.** The method provides no benefits, and leads to many unpleasant problems. The biggest problems are due to the fact that React treats a component defined inside another as a new component in every render. This makes it impossible for React to optimize the component.

## Further Reading for React
Official React Documentation: https://reactjs.org/docs/hello-world.html

Egghead.io Start learning React: https://egghead.io/courses/start-learning-react

Egghead.io Beginner's Guide to React: https://egghead.io/courses/the-beginner-s-guide-to-reactjs
