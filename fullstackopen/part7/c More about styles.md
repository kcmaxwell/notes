# More about styles

## Ready-made UI libraries

One approach to defining styles for an application is to use a ready-made "UI framework".

One of the first widely popular UI frameworks was the Bootstrap toolkit created by Twitter, which may still be the most popular framework. Recently, there has been a large increase in the number of new UI frameworks, so it is near impossible to create an exhaustive list of options.

Many UI frameworks provide developers of web applications with ready-made themes and "components" like buttons, menus, and tables. We write components in quotes because, in this context, we are not talking about React components. Usually, UI frameworks are used by including the CSS stylesheets and JavaScript code of the framework in the application.

Many UI frameworks have React-friendly versions where the framework's "components" have been transformed into React components. There are a few different React versions of Bootstrap like reactstrap and react-bootstrap.

Next, we will look at two UI frameworks, Bootstrap and MaterialUI. We will use both frameworks to add similar styles to the application we made in the React-router section of the course material.

## React Bootstrap

Let's start by taking a look at Bootstrap with the help of the react-bootstrap package.

Let's install the package with the command:
```
npm install react-bootstrap
```

Then, let's add a link for loading the CSS stylesheet for Bootstrap inside of the *head* tag in the *public/index.html* file of the application:
```
<head>
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"
    integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3"
    crossOrigin="anonymous"
  />
  // ...
</head>
```

In Bootstrap, all of the contents of the application are typically rendered inside a container. In practice, this is accomplished by giving the root `div` element of the application the `container` class attribute:
```
const App = () => {
  // ...

  return (
    <div className="container">      
      // ...
    </div>
  )
}
```

Next, let's make some changes to the *Notes* component so that it renders the list of notes as a table. React Bootstrap provides a built-in Table component for this purpose, so there is no need to define CSS classes separately.
```
const Notes = ({ notes }) => (
  <div>
    <h2>Notes</h2>
    <Table striped>      
      <tbody>
        {notes.map(note =>
          <tr key={note.id}>
            <td>
              <Link to={`/notes/${note.id}`}>
                {note.content}
              </Link>
            </td>
            <td>
              {note.user}
            </td>
          </tr>
        )}
      </tbody>
    </Table>
  </div>
)
```

Note that the React Bootstrap components have to be imported separately from the library as shown below:
```
import { Table } from 'react-bootstrap'
```

### Forms

Let's improve the form in the *Login* view with the help of Bootstrap forms.

React Bootstrap provides built-in components for creating forms:
```
import { Table, Form, Button } from 'react-bootstrap'

let Login = (props) => {
  // ...
  return (
    <div>
      <h2>login</h2>
      <Form onSubmit={onSubmit}>
        <Form.Group>
          <Form.Label>username:</Form.Label>
          <Form.Control
            type="text"
            name="username"
          />
          <Form.Label>password:</Form.Label>
          <Form.Control
            type="password"
          />
          <Button variant="primary" type="submit">
            login
          </Button>
        </Form.Group>
      </Form>
    </div>
)}
```

### Notification

Let's add a message for the notification when a user logs into the application. We will store it in the `message` variable in the *App* component's state:
```
const App = () => {
  const [notes, setNotes] = useState([
    // ...
  ])

  const [user, setUser] = useState(null)
  const [message, setMessage] = useState(null)
  const login = (user) => {
    setUser(user)
    setMessage(`welcome ${user}`)    
    setTimeout(() => {      
        setMessage(null)    
    }, 10000)  
  }
  // ...
}
```

We will render the message as a Bootstrap Alert component. Once again, the React Bootstrap library provides us with a matching React component:
```
<div className="container">
  {(message &&    
    <Alert variant="success">     
      {message}   
    </Alert>  
  )}  
  // ...
</div>
```

### Navigation structure

Lastly, let's alter the application's navigation menu to use Bootstrap's Navbar component. The React Bootstrap library provides us with matching built-in components.
```
<Navbar collapseOnSelect expand="lg" bg="dark" variant="dark">
  <Navbar.Toggle aria-controls="responsive-navbar-nav" />
  <Navbar.Collapse id="responsive-navbar-nav">
    <Nav className="me-auto">
      <Nav.Link href="#" as="span">
        <Link style={padding} to="/">home</Link>
      </Nav.Link>
      <Nav.Link href="#" as="span">
        <Link style={padding} to="/notes">notes</Link>
      </Nav.Link>
      <Nav.Link href="#" as="span">
        <Link style={padding} to="/users">users</Link>
      </Nav.Link>
      <Nav.Link href="#" as="span">
        {user
          ? <em style={padding}>{user} logged in</em>
          : <Link style={padding} to="/login">login</Link>
        }
      </Nav.Link>
    </Nav>
  </Navbar.Collapse>
</Navbar>
```

Bootstrap and a large majority of UI frameworks produce responsive designs, meaning that the resulting applications render well on a variety of different screen sizes.

## Material UI

As our second example, we will look into the MaterialUI React library, which implements the Material Design visual language developed by Google.

Install the library with the command:
```
npm install @mui/material @emotion/react @emotion/styled
```

Then add the following line to the *head* tag in the *public/index.html* file. The line loads Google's font Roboto.
```
<head>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" />
  // ...
</head>
```

Now let's use MaterialUI to do the same modifications to the code we did earlier with Bootstrap.

Render the contents of the whole application within a Container:
```
import { Container } from '@mui/material'

const App = () => {
  // ...
  return (
    <Container>
      // ...
    </Container>
  )
}
```

Let's start with the *Notes* component. We'll render the list of notes as a table:
```
const Notes = ({ notes }) => (
  <div>
    <h2>Notes</h2>

    <TableContainer component={Paper}>
      <Table>
        <TableBody>
          {notes.map(note => (
            <TableRow key={note.id}>
              <TableCell>
                <Link to={`/notes/${note.id}`}>{note.content}</Link>
              </TableCell>
              <TableCell>
                {note.user}
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  </div>
)
```

With MaterialUI, each component has to be imported separately. The import list for the notes page is quite long:
```
import {
  Container,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableRow,
  Paper,
} from '@mui/material'
```

### Form

Next, let's make the login form in the *Login* view better using the TextField and Button components:
```
const Login = (props) => {
  const navigate = useNavigate()

  const onSubmit = (event) => {
    event.preventDefault()
    props.onLogin('mluukkai')
    navigate('/')
  }

  return (
    <div>
      <h2>login</h2>
      <form onSubmit={onSubmit}>
        <div>
          <TextField label="username" />
        </div>
        <div>
          <TextField label="password" type='password' />
        </div>
        <div>
          <Button variant="contained" color="primary" type="submit">
            login
          </Button>
        </div>
      </form>
    </div>
  )
}
```

MaterialUI, unlike Bootstrap, does not provide a component for the form itself. The form here is an ordinary HTML form element.

Remember to import all the components used in the form.

### Notification

The notification displayed on login can be done using the Alert component, which is quite similar to Bootstrap's equivalent component:
```
<div>
  {(message &&    
    <Alert severity="success">      
      {message}    
    </Alert> 
  )}
</div>
```

### Navigation structure

We can implement navigation using the AppBar component.

Here is code built from the example code in the documentation:
```
<AppBar position="static">
  <Toolbar>
    <IconButton edge="start" color="inherit" aria-label="menu">
    </IconButton>
    <Button color="inherit">
      <Link to="/">home</Link>
    </Button>
    <Button color="inherit">
      <Link to="/notes">notes</Link>
    </Button>
    <Button color="inherit">
      <Link to="/users">users</Link>
    </Button>  
    <Button color="inherit">
      {user
        ? <em>{user} logged in</em>
        : <Link to="/login">login</Link>
      }
    </Button>                
  </Toolbar>
</AppBar>
```

We can make it look better by using component props to define how the root element of a MaterialUI component is rendered.

By defining:
```
<Button color="inherit" component={Link} to="/">
  home
</Button>
```

The `Button` component is rendered so that its root component is react-router-dom's `Link` which receives its path as the prop field `to`.

Here is our final code for the navigation bar:
```
<AppBar position="static">
  <Toolbar>
    <Button color="inherit" component={Link} to="/">
      home
    </Button>
    <Button color="inherit" component={Link} to="/notes">
      notes
    </Button>
    <Button color="inherit" component={Link} to="/users">
      users
    </Button>   
    {user
      ? <em>{user} logged in</em>
      : <Button color="inherit" component={Link} to="/login">
          login
        </Button>
    }                              
  </Toolbar>
</AppBar>
```

## Closing thoughts

The difference between react-bootstrap and MaterialUI is not big. It's up to personal preference which to use.

In the previous examples, we used the UI frameworks with the help of React-integration libraries.

Instead of using the React Bootstrap library, we could have just as well used Bootstrap directly by defining CSS classes for our application's HTML elements.

Instead of defining the table with the *Table* component:
```
<Table striped>
  // ...
</Table>
```

We could have used a regular HTML *table* and added the required CSS class:
```
<table className="table striped">
  // ...
</table>
```

In addition to making the frontend code more compact and readable, another benefit of using React UI framework libraries is that they include the JavaScript that is needed to make specific components work. Some Bootstrap components require a few JavaScript dependencies that we would prefer not to include in our React applications.

Some potential downsides to using UI frameworks through integration libraries instead of using the "directly" are that integration libraries may have unstable APIs and poor documentation.

There is also the question of whether or not UI framework libraries should be used in the first place. It is up to everyone to form their own opinion, but for people lacking knowledge in CSS and web design, they are very useful tools.

## Other UI frameworks

- https://bulma.io/
- https://ant.design/
- https://get.foundation/
- https://chakra-ui.com/
- https://tailwindcss.com/
- https://semantic-ui.com/
- https://mantine.dev/
- https://react.fluentui.dev/
- https://storybook.js.org
- https://www.primefaces.org/primereact/
- https://v2.grommet.io
- https://blueprintjs.com
- https://evergreen.segment.com
- https://www.radix-ui.com/
- https://react-spectrum.adobe.com/react-aria/index.html
- https://master.co/
- https://nextui.org/

## Styled components

There are also other ways of styling React applications that we have not yet taken a look at.

The styled components library offers an interesting approach for defining styles through tagged template literals that were introduced in ES6.

Let's make a few changes to the styles of our application with the help of styled components. First, install the package with the commmand:
```
npm install styled-components
```

Then, let's define two components with styles:
```
import styled from 'styled-components'

const Button = styled.button`
  background: Bisque;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid Chocolate;
  border-radius: 3px;
`

const Input = styled.input`
  margin: 0.25em;
`
```

The code above creates styled versions of the *button* and *input* HTML elements, and then assigns them to the *Button* and *Input* variables.

THe syntax for defining the styles is quite interesting, as the CSS rules are defined inside of backticks.

The styled components that we defined work exactly like regular *button* and *input* elements, and they can be used in the same way:
```
const Login = (props) => {
  // ...
  return (
    <div>
      <h2>login</h2>
      <form onSubmit={onSubmit}>
        <div>
          username:
          <Input />        
        </div>
        <div>
          password:
          <Input type='password' />        
        </div>
        <Button type="submit" primary=''>login</Button>      
      </form>
    </div>
  )
}
```

Let's create a few more components for styling that application which will be styled versions of *div* elements:
```
const Page = styled.div`
  padding: 1em;
  background: papayawhip;
`

const Navigation = styled.div`
  background: BurlyWood;
  padding: 1em;
`

const Footer = styled.div`
  background: Chocolate;
  padding: 1em;
  margin-top: 1em;
`
```

Let's use the components in our application:
```
const App = () => {
  // ...

  return (
    <Page>      
      <Navigation>        
        <Link style={padding} to="/">home</Link>
        <Link style={padding} to="/notes">notes</Link>
        <Link style={padding} to="/users">users</Link>
        {user
          ? <em>{user} logged in</em>
          : <Link style={padding} to="/login">login</Link>
        }
      </Navigation>      
      <Routes>
        <Route path="/notes/:id" element={<Note note={note} />} />  
        <Route path="/notes" element={<Notes notes={notes} />} />   
        <Route path="/users" element={user ? <Users /> : <Navigate replace to="/login" />} />
        <Route path="/login" element={<Login onLogin={login} />} />
        <Route path="/" element={<Home />} />      
      </Routes>

      <Footer>        
        <em>Note app, Department of Computer Science 2022</em>
      </Footer>    
    </Page>  
  )
}
```
