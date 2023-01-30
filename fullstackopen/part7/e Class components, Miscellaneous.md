# Class Components, Miscellaneous

## Class components

Before version 16.8 of React, the hook functionality was not available. When defining a component that uses state, one had to define it using JavaScript's Class syntax.

It is beneficial to be familiar with Class Components to some extent to be able to understand old React code.

Let's get to know the main features of Class Components by producing yet another very familiar anecdote application. We store the anecdotes in the file *db.json* using *json-server*. The contents of the file are can be seen here:
https://github.com/fullstack-hy/misc/blob/main/anecdotes.json

The initial version of the Class Component looks like this:
```
import React from 'react'

class App extends React.Component {
  constructor(props) {
    super(props)
  }

  render() {
    return (
      <div>
        <h1>anecdote of the day</h1>
      </div>
    )
  }
}

export default App
```

The component now has a constructor, in which nothing happens at the moment, and contains the method render. `render` defines how and what is rendered to the screen.

Let's define a state for the list of anecdotes and the currently-visible anecdote. In contrast to when using the useState hook, Class Components only contain one state. So if the state is made up of multiple "parts", they should be stored as properties of the state. The state is initialized in the constructor:
```
class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {      anecdotes: [],      current: 0    }  }

  render() {
    if (this.state.anecdotes.length === 0) {
        return <div>no anecdotes...</div>
    }
    return (
      <div>
        <h1>anecdote of the day</h1>
        <div>
          {this.state.anecdotes[this.state.current].content}
        </div>
        <button>next</button>
      </div>
    )
  }
}
```

The component state is in the instance variable `this.state`. The state is an object having two properties. *this.state.anecdotes* is the list of anecdotes and *this.state.current* is the index of the currently-shown anecdote.

In Functional components, the right place for fetching data from a server is inside an effect hook, which is executed when a component renders or less frequently if necessary.

The lifecycle methods of Class Components offer corresponding functionality. The correct place to trigger the fetching of data from a server is inside the lifecycle method `componentDidMount`, which is executed once right after the first time a component renders:
```
class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      anecdotes: [],
      current: 0
    }
  }

  componentDidMount = () => {
    axios.get('http://localhost:3001/anecdotes').then(response => {
        this.setState({ anecdotes: response.data })
    })
  }

  // ...
}
```

The callback function of the HTTP request updates the component state using the method setState. The method only touches the keys that have been defined in the object passed to the method as an argument. The value for the key *current* remains unchanged.

Calling the method setState always triggers the rerender of the Class Compponent, i.e. calls the `render` function.

We'll finish off the component with the ability to change the shown anecdote. The following is the code for the entire component with the addition highlighted:
```
class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      anecdotes: [],
      current: 0
    }
  }

  componentDidMount = () => {
    axios.get('http://localhost:3001/anecdotes').then(response => {
      this.setState({ anecdotes: response.data })
    })
  }

  handleClick = () => {
    const current = Math.floor(
      Math.random() * this.state.anecdotes.length
    )
    this.setState({ current })
  }

  render() {
    if (this.state.anecdotes.length === 0 ) {
      return <div>no anecdotes...</div>
    }

    return (
      <div>
        <h1>anecdote of the day</h1>
        <div>{this.state.anecdotes[this.state.current].content}</div>
        <button onClick={this.handleClick}>next</button>
      </div>
    )
  }
}
```

For comparison, here is the same component as a Functional component:
```
const App = () => {
  const [anecdotes, setAnecdotes] = useState([])
  const [current, setCurrent] = useState(0)

  useEffect(() =>{
    axios.get('http://localhost:3001/anecdotes').then(response => {
      setAnecdotes(response.data)
    })
  },[])

  const handleClick = () => {
    setCurrent(Math.round(Math.random() * (anecdotes.length - 1)))
  }

  if (anecdotes.length === 0) {
    return <div>no anecdotes...</div>
  }

  return (
    <div>
      <h1>anecdote of the day</h1>
      <div>{anecdotes[current].content}</div>
      <button onClick={handleClick}>next</button>
    </div>
  )
}
```

The biggest difference between Functional components and Class components is mainly that the state of a Class component is a single object, and that the state is updated using the method `setState`, while in Functional components, the state can consist of multiple different variables, with all of them having their own update function.

In some more advanced use cases, the effect hook offers a considerably better mechanism for controlling side effects compared to the lifecycle methods of Class components.

A notable benefit of using Functional components is not having to deal with the self-referencing `this` reference of the JavaScript class.

When writing fresh code, use Functional components if the React version is 16.8 or higher. There is no need to rewrite all old React code as Functional components, however.

## Organization of code in React application

In the exercises, we followed the principle by which components were placed in the directory *components*, reducers were placed in the directory *reducers*, and the code responsible for communicating with the server was placed in the directory *services*.

This works for a smaller application, but as the amount of components increases, better solutions are needed. There is no one correct way to organize a project though. Here is an article that discusses this issue:
https://david-gilbertson.medium.com/the-100-correct-way-to-structure-a-react-app-or-why-theres-no-such-thing-3ede534ef1ed

## Frontend and backend in the same repository

In previous exercises, we created the frontend and backend into separate repositories. This is a typical approach. However, we did the deployment by copying the bundled frontend code into the backend repository. A possibly better approach would have been to deploy the frontend code separately. 

Sometimes, there may be a situation where the entire application is to be put into a single repository. In this case, a common approach is to put the *package.json* and *webpack.config.js* in the root directory, as well as place the frontend and backend code into their own directories, e.g. *client* and *server*.

The following repository provides one possible starting point for the organization of "single repository code":
https://github.com/fullstack-hy2020/create-app

## Changes on the server

