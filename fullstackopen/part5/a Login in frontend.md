# Login in frontend

## Handling login

A login form has been added to the frontend of the notes app. The code of the *App* component now looks as follows:
```
const App = () => {
  const [notes, setNotes] = useState([]) 
  const [newNote, setNewNote] = useState('')
  const [showAll, setShowAll] = useState(true)
  const [errorMessage, setErrorMessage] = useState(null)
  const [username, setUsername] = useState('')   
  const [password, setPassword] = useState('') 

  useEffect(() => {
    noteService
      .getAll().then(initialNotes => {
        setNotes(initialNotes)
      })
  }, [])

  // ...

  const handleLogin = (event) => {    
    event.preventDefault()    
    console.log('logging in with', username, password)  
  }

  return (
    <div>
      <h1>Notes</h1>

      <Notification message={errorMessage} />

      <form onSubmit={handleLogin}>        
        <div>          
          username            
            <input            
            type="text"            
            value={username}            
            name="Username"            
            onChange={({ target }) => setUsername(target.value)}          
            />        
        </div>        
        <div>          
          password            
            <input            
            type="password"            
            value={password}            
            name="Password"            
            onChange={({ target }) => setPassword(target.value)}          
            />        
        </div>        
        <button type="submit">login</button>      
      </form>

      // ...
    </div>
  )
}

export default App
```

The login form is handled the same as we handled forms in part 2. The app state has fields for *username* and *password* to store the data from the form. The form fields have event handlers, which synchronize changes in the field to the state of the *App* component. The event handlers are simple: An object is given to them as a parameter, and they destructure the field *target* from the object and save its value to the state.

The method `handleLogin`, which is responsible for handling the data in the form, will be described later.

Logging in is done by sending an HTTP POST request to the server address *api/login*. We will separate the code responsible for this request into its own module, to the file *services/login.js*:
```
import axios from 'axios'
const baseUrl = '/api/login'

const login = async credentials => {
  const response = await axios.post(baseUrl, credentials)
  return response.data
}

export default { login }
```

The `handleLogin` method can be implemented as follows:
```
import loginService from './services/login'
const App = () => {
  // ...
  const [username, setUsername] = useState('') 
  const [password, setPassword] = useState('') 
  const [user, setUser] = useState(null)  

  const handleLogin = async (event) => {    
    event.preventDefault()        

    try {      
      const user = await loginService.login({        
        username, password,      
      })      
      setUser(user)      
      setUsername('')      
      setPassword('')    
    } catch (exception) {      
      setErrorMessage('Wrong credentials')      
      setTimeout(() => {        
        setErrorMessage(null)      
      }, 5000)    
    }  
  }

  // ...
}
```

If the login is successful, the form fields are emptied *and* the server response (including a *token* and the user details) is saved to the *user* field of the application's state.

If the login fails, or running the function `loginService.login` results in an error, the user is notified.

## Creating new notes

To create new notes with our current backend, we need to add the token of the logged-in user to the Authorization header of the HTTP request. The *noteService* module changes like so:
```
import axios from 'axios'
const baseUrl = '/api/notes'

let token = null

const setToken = newToken => {  
  token = `bearer ${newToken}`
}

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

const create = async newObject => {
  const config = {    
    headers: { Authorization: token },  
  }
  const response = await axios.post(baseUrl, newObject, config)  return response.data
}

const update = (id, newObject) => {
  const request = axios.put(`${ baseUrl }/${id}`, newObject)
  return request.then(response => response.data)
}

export default { getAll, create, update, setToken }
```

The noteService module contains a private variable `token`. Its value can be changed with a function `setToken`, which is exported by the module. `create` sets the token to the *Authorization* header. The header is given to axios as the third parameter of the *post* method. We only need to add the line `noteService.setToken(user.token)` to the `handleLogin` function.

## Saving the token to the browser's local storage

Our application has a flaw: when the page is rerendered, the user's login information disappears. This also slows down development. For example, when testing creating new notes, we have to login every time.

This problem is easily solved by saving the login details to local storage. Local Storage is a key-value database in the browser.

A *value* corresponding to a certain *key* is saved to the database with the method setItem. For example:
```
window.localStorage.setItem('name', 'juha tauriainen')
```

The value of a key can be found with the method getItem:
```
window.localStorage.getItem('name')
```

The method removeItem removes a key.

Values in the local storage are persisted even when the page is re-rendered. The storage is origin-specific, so each web application has its own storage.

Values saved to the storage are DOMstrings, so we cannot save a JavaScript object as it is. The object has to be parsed to JSON first, with the method `JSON.stringify`. Correspondingly, when a JSON object is read from the local storage, it has to be parsed back to JavaScript with `JSON.parse`.

Using local storage, we can change the login method like so:
```
const handleLogin = async (event) => {
  event.preventDefault()
  try {
    const user = await loginService.login({
      username, password,
    })

    window.localStorage.setItem(        
      'loggedNoteappUser', JSON.stringify(user)      
    )       
    noteService.setToken(user.token)
    setUser(user)
    setUsername('')
    setPassword('')
  } catch (exception) {
    // ...
  }
}
```

You can view the details of a logged-in user after being saved using `window.localStorage` in the console. You can also inspect it using the developer tools. On Chrome, go to the *Application* tab and select *Local Storage*. On Firefox, go to the *Storage* tab and select *Local Storage*.

We still have to modify our application so that when we enter the page, the application checks if user details of a logged-in user can already be found on the local storage. If they can, the details are saved to the state of the application and to *noteService*.

The right way to do this is with an effect hook. We can have multiple effect hooks, so let's create a second one to handle the first loading of the page:
```
const App = () => {
  const [notes, setNotes] = useState([]) 
  const [newNote, setNewNote] = useState('')
  const [showAll, setShowAll] = useState(true)
  const [errorMessage, setErrorMessage] = useState(null)
  const [username, setUsername] = useState('') 
  const [password, setPassword] = useState('') 
  const [user, setUser] = useState(null) 

  useEffect(() => {
    noteService
      .getAll().then(initialNotes => {
        setNotes(initialNotes)
      })
  }, [])

  useEffect(() => {    
    const loggedUserJSON = window.localStorage.getItem('loggedNoteappUser')    
    if (loggedUserJSON) {      
      const user = JSON.parse(loggedUserJSON)      
      setUser(user)      
      noteService.setToken(user.token)    
    }  
  }, [])

  // ...
}
```

The empty array as the second parameter of the effect ensures that the effect is only executed when the component is rendered for the first time.

Logout is not yet implemented, but can be done using the console with the command:
```
window.localStorage.removeItem('loggedNoteappUser')
```

Or with the command to empty *localstorage* completely:
```
window.localStorage.clear()
```

## A note on using local storage

At the end of the last part, we mentioned that the challenge of token-based authentication is how to cope with the situation when the API access of the token holder needs to be revoked.

There are two solutions. One is to limit the validity period of a token, forcing the user to re-login once the token expires. The other is to save the validity information of each token to the backend database. This is also called a *server-side session*.

No matter how the validity of tokens is checked, saving a token in local storage might contain a security risk if the application has a security vulnerability that allows Cross Site Scripting (XSS) attacks. An XSS attack is possible if the application would allow a user to inject arbitrary JavaScript code that the app would then execute. When using React sensibly, it should not be possible, since React sanitizes all text that it renders, meaning that it is not executing the rendered content as JavaScript.

It has been suggested that the identity of a signed-in user should be saved as httpOnly cookies, so that JavaScript code could not have any access to the token. The drawback of this is that it makes implementing SPA apps a bit more complex. You would at least a separate page for logging in.

However, the use of httpOnly cookies does not guarantee anything. It has been suggested that httpOnly cookies are no safer than the use of local storage.

No matter which solution is used, the key is to minimize the risk of XSS attacks.
