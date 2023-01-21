# React-router

## Application navigation structure

Following part 6, we return to React without Redux.

It is very common for web applications to have a navigation bar, which enables switching the view of the application.

In an old school web app, changing the page shown by the application would be accomplished by the browser making an HTTP GET request to the server and rendering the HTML representing the view that was returned.

In single-page apps, we are, in reality, always on the same page. The JavaScript code run by the browser creates an illusion of different pages. If HTTP requests are made when switching views, they are only for fetching JSON-formatted data, which the new view might require for it to be shown.

The navigation bar and an application containing multiple views are very easy to implement using React. Here is one way:
```
import { useState }  from 'react'
import ReactDOM from 'react-dom/client'

const Home = () => (
  <div> <h2>TKTL notes app</h2> </div>
)

const Notes = () => (
  <div> <h2>Notes</h2> </div>
)

const Users = () => (
  <div> <h2>Users</h2> </div>
)

const App = () => {
  const [page, setPage] = useState('home')

  const toPage = (page) => (event) => {
    event.preventDefault()
    setPage(page)
  }

  const content = () => {
    if (page === 'home') {
      return <Home />
    } else if (page === 'notes') {
      return <Notes />
    } else if (page === 'users') {
      return <Users />
    }
  }

  const padding = {
    padding: 5
  }

  return (
    <div>
      <div>
        <a href="" onClick={toPage('home')} style={padding}>
          home
        </a>
        <a href="" onClick={toPage('notes')} style={padding}>
          notes
        </a>
        <a href="" onClick={toPage('users')} style={padding}>
          users
        </a>
      </div>

      {content()}
    </div>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

Each view is implemented as its own component. We store the view component information in the application state called *page*. This information tells us which component, representing a view, should be shown below the menu bar.

However, this method is not very optimal. The address stays the same, even though at times, we are in different views. Each view should preferably have its own address, e.g. to make bookmarking possible. The *back* button doesn't work as expected for our application either, meaning that *back* doesn't move you to the previously displayed view of the application, but somewhere completely different. If the application were to grow even bigger and we wanted to, for example, add separate views for each user and note, then this self-made *routing*, which means the navigation management of the application, would get overly complicated.

## React Router

Luckily, React has the React Router library, which provides an excellent solution for managing navigation in a React application.

Let's change the above application to use React Router. First, we install React Router:
```
npm install react-router-dom
```

The routing provided by React Router is enabled by changing the application as follows:
```
import {
  BrowserRouter as Router,
  Routes, Route, Link
} from "react-router-dom"

const App = () => {

  const padding = {
    padding: 5
  }

  return (
    <Router>
      <div>
        <Link style={padding} to="/">home</Link>
        <Link style={padding} to="/notes">notes</Link>
        <Link style={padding} to="/users">users</Link>
      </div>

      <Routes>
        <Route path="/notes" element={<Notes />} />
        <Route path="/users" element={<Users />} />
        <Route path="/" element={<Home />} />
      </Routes>

      <div>
        <i>Note app, Department of Computer Science 2022</i>
      </div>
    </Router>
  )
}
```

Routing, or the conditional rendering of components *based on the URL* in the browser, is used by placing components as children of the *Router* component, meaning inside *Router* tags.

Notice that, even though the component is referred to by the name *Router*, we are talking about *BrowserRouter*, because here, the import happens by renaming the imported object.

According to the v5 docs: BrowserRouter is a Router that uses the HTML5 history API (pushState, replaceState, and the popState event) to keep your UI in sync with the URL.

Normally, the browser loads a new page when the URL in the address bar changes. However, with the help of the HTML5 history API, *BrowserRouter* enables us to use the URL in the address bar of the browser for internal "routing" in a React application. So even if the URL in the address bar changes, the content of the page is only manipulated using JavaScript, and the browser will not load new content from the server. Using the back and forward actions, as well as making bookmarks, is still logical like on a traditional web page.

Inside the router, we define *links* that modify the address bar with the help of the Link component.For example,
```
<Link to="/notes">notes</Link>
```
creates a link in the application with the text *notes*, which when clicked, changes the URL in the address bar to */notes*.

Components rendered based on the URL of the browser are defined with the help of the component *Route*. For example,
```
<Route path="/notes" element={<Notes />} />
```
defines that, if the browser address is */notes*, we render the *Notes* component.

We wrap the components to be rendered based on the URL with a *Routes* component. The *Routes* works by rendering the first component whose *path* matches the URL in the browser's address bar.

## Parameterized route

We will continue by examining a modified version of the above example, available here:
https://github.com/fullstack-hy2020/misc/blob/master/router-app-v1.js

The application now contains five different views whose display is controlled by the router. In addition to the components from the previous example (*Home*, *Notes*, and *Users*), we have *Login* representing the login view, and *Note* representing the view of a single note.

*Notes* renders the list of notes passed to it as props in such a way that the name of each note is clickable.

The ability to click a name is implemented with the component *Link*, and clicking the name of a note whose id is 3 would trigger an event that changes the address of the browser into *notes/3*:
```
const Notes = ({notes}) => (
  <div>
    <h2>Notes</h2>
    <ul>
      {notes.map(note =>
        <li key={note.id}>
          <Link to={`/notes/${note.id}`}>{note.content}</Link>
        </li>
      )}
    </ul>
  </div>
)
```

We define parameterized URLs in the routing in *App* as follows:
```
<Router>
  // ...

  <Routes>
    <Route path="/notes/:id" element={<Note notes={notes} />} />    <Route path="/notes" element={<Notes notes={notes} />} />   
    <Route path="/users" element={user ? <Users /> : <Navigate replace to="/login" />} />
    <Route path="/login" element={<Login onLogin={login} />} />
    <Route path="/" element={<Home />} />      
  </Routes>
</Router>
```

We define the route rendering a specific note "express style" by marking the parameter with a colon. When a browser navigates to the URL for a specific note, for example, */notes/3*, we render the *Note* component:
```
import {
  // ...
  useParams} from "react-router-dom"

const Note = ({ notes }) => {
  const id = useParams().id  const note = notes.find(n => n.id === Number(id)) 
  return (
    <div>
      <h2>{note.content}</h2>
      <div>{note.user}</div>
      <div><strong>{note.important ? 'important' : ''}</strong></div>
    </div>
  )
}
```

The *Note* component receives all of the notes as props *notes*, and it can access the URL parameter (the id of the note to be displayed) with the *useParams* function of the React Router.

## useNavigate

We have also implemented a simple login function in our application. If a user is logged in, information about a logged-in user is saved to the *user* field of the state of the *App* component.

The option to navigate to the *Login* view is rendered conditionally in the menu:
```
<Router>
  <div>
    <Link style={padding} to="/">home</Link>
    <Link style={padding} to="/notes">notes</Link>
    <Link style={padding} to="/users">users</Link>
    {user      
      ? <em>{user} logged in</em>      
      : <Link style={padding} to="/login">login</Link>    
    }  
  </div>

  // ...
</Router>
```

So if the user is already logged in, instead of displaying the link *Login*, we show the username of the user.

The code of the component handling the login functionality is as follows:
```
import {
  // ...
  useNavigate
} from 'react-router-dom'

const Login = (props) => {
  const navigate = useNavigate()
  const onSubmit = (event) => {
    event.preventDefault()
    props.onLogin('mluukkai')
    navigate('/')  }

  return (
    <div>
      <h2>login</h2>
      <form onSubmit={onSubmit}>
        <div>
          username: <input />
        </div>
        <div>
          password: <input type='password' />
        </div>
        <button type="submit">login</button>
      </form>
    </div>
  )
}
```

With the *useNavigate* function of React Router, the browser's URL can be changed programmatically.

With user login, we call `navigate('/')`, which causes the browser's URL to change to `/`, and the application renders the corresponding component *Home*.

Both useParams and useNavigate are hook functions, just like useState and useEffect. As you remember, there are some rules to using hook functions. Create-react-app has been configured to warn you if you break these rules, for example, by calling a hook function from a conditional statement.

## redirect

There is one more interesting detail about the *Users* route:
```
<Route path="/users" element={user ? <Users /> : <Navigate replace to="/login" />} />
```

If a user isn't logged in, the *Users* component is not rendered. Instead, the user is *redirected* using the component *Navigate* to the login view.

Here is the *App* component in its entirety:
```
const App = () => {
  const [notes, setNotes] = useState([
    // ...
  ])

  const [user, setUser] = useState(null) 

  const login = (user) => {
    setUser(user)
  }

  const padding = {
    padding: 5
  }

  return (
    <div>
      <Router>
        <div>
          <Link style={padding} to="/">home</Link>
          <Link style={padding} to="/notes">notes</Link>
          <Link style={padding} to="/users">users</Link>
          {user
            ? <em>{user} logged in</em>
            : <Link style={padding} to="/login">login</Link>
          }
        </div>

        <Routes>
          <Route path="/notes/:id" element={<Note notes={notes} />} />  
          <Route path="/notes" element={<Notes notes={notes} />} />   
          <Route path="/users" element={user ? <Users /> : <Navigate replace to="/login" />} />
          <Route path="/login" element={<Login onLogin={login} />} />
          <Route path="/" element={<Home />} />      
        </Routes>
      </Router>      
      <footer>
        <br />
        <em>Note app, Department of Computer Science 2022</em>
      </footer>
    </div>
  )
}
```

We define an element common for modern web apps called *footer*, which defines the part at the bottom of the screen, outside of the *Router*, so that it is shown regardless of the component shown in the routed part of the application.

## Parameterized route revisited

Our application has a flaw. The `Note` component receives all of the notes, even though it only displays the one whose id matches the url parameter:
```
const Note = ({ notes }) => { 
  const id = useParams().id
  const note = notes.find(n => n.id === Number(id))
  // ...
}
```

We would like to be able to have the `Note` component receive only the note that it needs to display:
```
const Note = ({ note }) => {
  return (
    <div>
      <h2>{note.content}</h2>
      <div>{note.user}</div>
      <div><strong>{note.important ? 'important' : ''}</strong></div>
    </div>
  )
}
```

One way to do this would be to use React Router's *useMatch* hook to figure out the id of the note to be displayed in the `App` component.

It is not possible to use the *useMatch* hook in the component which defines the routed part of the application. Let's move the use of the `Router` components from `App`:
```
ReactDOM.createRoot(document.getElementById('root')).render(
  <Router>    
    <App />
  </Router>
)
```

The `App` component becomes:
```
import {
  // ...
  useMatch
} from "react-router-dom"

const App = () => {
  // ...

  const match = useMatch('/notes/:id')  
  const note = match     
    ? notes.find(note => note.id === Number(match.params.id))    
    : null

  return (
    <div>
      <div>
        <Link style={padding} to="/">home</Link>
        // ...
      </div>

      <Routes>
        <Route path="/notes/:id" element={<Note note={note} />} />        
        <Route path="/notes" element={<Notes notes={notes} />} />   
        <Route path="/users" element={user ? <Users /> : <Navigate replace to="/login" />} />
        <Route path="/login" element={<Login onLogin={login} />} />
        <Route path="/" element={<Home />} />      
      </Routes>   

      <div>
        <em>Note app, Department of Computer Science 2022</em>
      </div>
    </div>
  )
}  
```

Every time the component is rendered, so practically every time the browser's URL changes, the following command is executed:
```
const match = useMatch('/notes/:id')
```

If the URL matches `/notes/:id`, the match variable will contain an object from which we can access the parameterized part of the path, the id of the note to be displayed, and we can then fetch the correct note to display.
