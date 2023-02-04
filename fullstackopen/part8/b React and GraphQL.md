# React and GraphQL

We will next implement a React app which uses the GraphQL server we created.

In theory, we could use GraphQL with HTTP POST requests. The communication works by sending HTTP POST requests to http://localhost:4000/graphql. The query itself is a string sent as the value of the key *query*.

We could take care of the communication between the React app and GraphQL by using Axios. However, most of the time, it is not sensible to do so. It is a better idea to use a higher-order library capable of abstracting the unnecessary details of the communication.

At the moment, there are two good options: *Relay* by Facebook, and *Apollo Client*, which is the client side of the same library we used in the previous section. Apollo is the most popular of the two, and we will use it in this section as well.

## Apollo Client

Let us create a new React app, and continue by installing dependencies required by React client:
```
npm install @apollo/client graphql
```

We'll start with the following code for our application:
```
import ReactDOM from 'react-dom/client'
import App from './App'

import { ApolloClient, InMemoryCache, gql } from '@apollo/client'

const client = new ApolloClient({
  uri: 'http://localhost:4000',
  cache: new InMemoryCache(),
})

const query = gql`
  query {
    allPersons {
      name,
      phone,
      address {
        street,
        city
      }
      id
    }
  }
`

client.query({ query })
  .then((response) => {
    console.log(response.data)
  })


ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

The beginning of the code creates a new client object, which is then used to send a query to the server. The server's response is then printed to the console.

The application can communicate with a GraphQL server using the `client` object. The client can be made accessible for all components of the application by wrapping the *App* component with *ApolloProvider*:
```
import ReactDOM from 'react-dom/client'
import App from './App'

import {
  ApolloClient,
  ApolloProvider,  InMemoryCache,
} from '@apollo/client'

const client = new ApolloClient({
  uri: 'http://localhost:4000',
  cache: new InMemoryCache(),
})

ReactDOM.createRoot(document.getElementById('root')).render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>)
```

## Making queries

We are ready to implement the main view of the application, which shows a list of persons names and phone numbers.

Apollo Client offers a few alternatives for making queries. Currently, the use of the hook function useQuery is the dominant practice.

The query is made by the *App* component, the code of which is as follows:
```
import { gql, useQuery } from '@apollo/client'

const ALL_PERSONS = gql`
query {
  allPersons {
    name
    phone
    id
  }
}
`

const App = () => {
  const result = useQuery(ALL_PERSONS)

  if (result.loading) {
    return <div>loading...</div>
  }

  return (
    <div>
      {result.data.allPersons.map(p => p.name).join(', ')}
    </div>
  )
}

export default App
```

When called, `useQuery` makes the query it receives as a parameter. It returns an object with multiple fields. The field *loading* is true if the query has not received a response yet. We use the field to return a *loading...* div early. When a response is received, the result of the *allPersons* query can be found from the *data* field, and we can render the list of names to the screen.

Let's separate displaying the list of persons into its own component:
```
const Persons = ({ persons }) => {
  return (
    <div>
      <h2>Persons</h2>
      {persons.map(p =>
        <div key={p.name}>
          {p.name} {p.phone}
        </div>  
      )}
    </div>
  )
}
```

The *App* component still makes the query, and passes the result to the new component to be rendered:
```
const App = () => {
  const result = useQuery(ALL_PERSONS)

  if (result.loading)  {
    return <div>loading...</div>
  }

  return (
    <Persons persons={result.data.allPersons}/>
  )
}
```

## Named queries and variables

Let's implement functionality for viewing the address details of a person. The *findPerson* query is well-suited for this.

The queries we did in the last chapter had the parameter hardcoded into the query. When we do queries programmatically, we must be able to give them parameters dynamically.

GraphQL variables are well-suited for this. To be able to use variables, we must also name our queries.

A good format for the query is this:
```
query findPersonByName($nameToSearch: String!) {
  findPerson(name: $nameToSearch) {
    name
    phone 
    address {
      street
      city
    }
  }
}
```

The name of the query is *findPersonByName*, and it is given a string *$nameToSearch* as a parameter. 

It is also possible to do queries with Apollo Explorer. The parameters are given in *Variables*, in the bottom left panel.

The `useQuery` hook is well-suited for situations where the query is done when the component is rendered. However, we now want to make the query only when a user wants to see the details of a specific person, so the query is done only as required.

One possibility for this kind of situation is the hook function `useLazyQuery`, which would make it possible to define a query which is executed *when* the user wants to see the detailed information of a person.

However, in our case, we can stick to `useQuery` and use the option skip, which makes it possible to do the query only if a set condition is true.

The solution is as follows:
```
import { useState } from 'react'
import { gql, useQuery } from '@apollo/client'

const FIND_PERSON = gql`
  query findPersonByName($nameToSearch: String!) {
    findPerson(name: $nameToSearch) {
      name
      phone
      id
      address {
        street
        city
      }
    }
  }
`

const Person = ({ person, onClose }) => {
  return (
    <div>
      <h2>{person.name}</h2>
      <div>
        {person.address.street} {person.address.city}
      </div>
      <div>{person.phone}</div>
      <button onClick={onClose}>close</button>
    </div>
  )
}

const Persons = ({ persons }) => {
  const [nameToSearch, setNameToSearch] = useState(null)
  const result = useQuery(FIND_PERSON, {
    variables: { nameToSearch },
    skip: !nameToSearch,
  })

  if (nameToSearch && result.data) {
    return (
      <Person
        person={result.data.findPerson}
        onClose={() => setNameToSearch(null)
      />
    )
  }

  return (
    <div>
      <h2>Persons</h2>
      {persons.map((p) => (
        <div key={p.name}>
          {p.name} {p.phone} 
          <button onClick={() => setNameToSearch(p.name)}>
            show address
          </button>
        </div>
      ))}
    </div>
  )
}

export default Persons
```

When the button *show address* of a person is pressed, the name of the person is set to state *nameToSearch*:
```
<button onClick={() => setNameToSearch(p.name)}>
  show address
</button>
```

This causes the component to re-render itself. On render, the query *FIND_PERSON* that fetches the detailed information of a user is executed *if the variable nameToSearch* has a value:
```
const result = useQuery(FIND_PERSON, {
  variables: { nameToSearch },
  skip: !nameToSearch,
})
```

When user is not interested in seeing the detaild info of any person, the state variable *nameToSearch* is null, and the query is not executed.

If the state *nameToSearch* has a value, and the query result is ready, the component *Person* renders the detailed info of a person.

When a user wants to return to the persons list, the *nameToSearch* state is set to `null`.

## Cache

When we do multiple queries, for example with the address details of Arto Hellas, we notice something interesting: the query to the backend is done only the first time around. After this, despite the same query being done again by the code, the query is not sent to the backend.

Apollo Client saves the response of queries to the cache. To optimize performance, if the response to a query is already in the cache, the query is not sent to the server at all.

## Doing mutations

Let's implement functionality for adding new persons.

In the previous chapter, we hardcoded the parameters for mutations. Now, we need a version of the addPerson mutation which uses variables:
```
const CREATE_PERSON = gql`
mutation createPerson($name: String!, $street: String!, $city: String!, $phone: String) {
  addPerson(
    name: $name,
    street: $street,
    city: $city,
    phone: $phone
  ) {
    name
    phone
    id
    address {
      street
      city
    }
  }
}
`
```

The hook function *useMutation* provides the functionality for making mutations.

Let's create a new component for adding a new person to the directory:
```
import { useState } from 'react'
import { gql, useMutation } from '@apollo/client'

const CREATE_PERSON = gql`
  // ...
`

const PersonForm = () => {
  const [name, setName] = useState('')
  const [phone, setPhone] = useState('')
  const [street, setStreet] = useState('')
  const [city, setCity] = useState('')

  const [ createPerson ] = useMutation(CREATE_PERSON)
  const submit = (event) => {
    event.preventDefault()

    createPerson({  variables: { name, phone, street, city } })
    setName('')
    setPhone('')
    setStreet('')
    setCity('')
  }

  return (
    <div>
      <h2>create new</h2>
      <form onSubmit={submit}>
        <div>
          name <input value={name}
            onChange={({ target }) => setName(target.value)}
          />
        </div>
        <div>
          phone <input value={phone}
            onChange={({ target }) => setPhone(target.value)}
          />
        </div>
        <div>
          street <input value={street}
            onChange={({ target }) => setStreet(target.value)}
          />
        </div>
        <div>
          city <input value={city}
            onChange={({ target }) => setCity(target.value)}
          />
        </div>
        <button type='submit'>add!</button>
      </form>
    </div>
  )
}

export default PersonForm
```

We can define mutation functions using the `useMutation` hook. The hook returns an *array*, the first element of which contains the function to cause the mutation. The query variables receive values when the query is made.

New persons are added just fine, but the screen is not updated. This is because Apollo Client cannot automatically update the cache of an application, so it still contains the state from before the mutation. We could update the screen by reloading the page, as the cache is emptied when the page is reloaded. However, there must be a better way to do this.

## Updating the cache

There are a few different solutions for this. One way is to make the query for all persons poll the server, or make the query repeatedly.

Let's set the query to poll every two seconds:
```
const App = () => {
  const result = useQuery(ALL_PERSONS, {
    pollInterval: 2000  })

  if (result.loading)  {
    return <div>loading...</div>
  }

  return (
    <div>
      <Persons persons = {result.data.allPersons}/>
      <PersonForm />
    </div>
  )
}

export default App
```

The solution is simple, and every time a user adds a new person, it appears immediately on the screens of all users.

The bad side of the solution is all the pointless web traffic.

Another easy way to keep the cache in sync is to use the `useMutation` hook's refetchQueries parameter to define that the query fetching all persons is done again whenever a new person is created.
```
const ALL_PERSONS = gql`
  query  {
    allPersons  {
      name
      phone
      id
    }
  }
`

const PersonForm = (props) => {
  // ...

  const [ createPerson ] = useMutation(CREATE_PERSON, {
    refetchQueries: [ { query: ALL_PERSONS } ]  })
```

The pros and cons of this solution are almost opposite of the previous one's. There is no extra web traffic, because queries are not done just in case. However, if one user now updates the state of the server, the changes do not show to other users immediately.

If you want to do multiple queries, you can pass multiple objects inside refetchQueries. This will allow you to update different parts of your app at the same time. Here is an example:
```
    const [ createPerson ] = useMutation(CREATE_PERSON, {
    refetchQueries: [ { query: ALL_PERSONS }, { query: OTHER_QUERY }, { query: ... } ] // pass as many queries as you need
  })
```

There are other ways to update the cache. More about those later in this part.

At the moment, queries and components are defined in the same place in our code. Let's separate the query definitions into their own file *queries.js*:
```
import { gql } from '@apollo/client'

export const ALL_PERSONS = gql`
  query {
    // ...
  }
`
export const FIND_PERSON = gql`
  query findPersonByName($nameToSearch: String!) {
    // ...
  }
`

export const CREATE_PERSON = gql`
  mutation createPerson($name: String!, $street: String!, $city: String!, $phone: String) {
    // ...
  }
`
```

Each component then imports the queries it needs:
```
import { ALL_PERSONS } from './queries'

const App = () => {
  const result = useQuery(ALL_PERSONS)
  // ...
}
```

## Handling mutation errors

Trying to create a person with invalid data causes an error. We should handle the exception. We can register an error handler function to the mutation using the `useMutation` hook's `onError` option.

Let's register the mutation with an error handler which uses the `setError` function it receives as a parameter to set an error message:
```
const PersonForm = ({ setError }) => {
  // ... 

  const [ createPerson ] = useMutation(CREATE_PERSON, {
    refetchQueries: [  {query: ALL_PERSONS } ],
    onError: (error) => {
      const errors = error.graphQLErrors[0].extensions.error.errors
      const messages = Object.values(errors).map(e => e.message).join('\n')
      setError(messages)
    }
  })

  // ...
}
```

We can then render the error message on the screen as necessary:
```
const App = () => {
  const [errorMessage, setErrorMessage] = useState(null)
  const result = useQuery(ALL_PERSONS)

  if (result.loading)  {
    return <div>loading...</div>
  }

  const notify = (message) => {
    setErrorMessage(message)
    setTimeout(() => {
      setErrorMessage(null)
    }, 10000)
  }

  return (
    <div>
      <Notify errorMessage={errorMessage} /> 
      <Persons persons = {result.data.allPersons} />
      <PersonForm setError={notify} />
    </div>
  )
}

const Notify = ({errorMessage}) => {
  if ( !errorMessage ) {
    return null
  }
  return (
    <div style={{color: 'red'}}>
    {errorMessage}
    </div>
  )
}
```

Now, the user is informed about an error with a simple notification.

## Updating a phone number

Let's add the possibility to change the phone numbers of persons to our application. The solution is almost identical to the one we used for adding new persons.

Again, the mutation requires parameters:
```
export const EDIT_NUMBER = gql`
  mutation editNumber($name: String!, $phone: String!) {
    editNumber(name: $name, phone: $phone) {
      name
      phone
      address {
        street
        city
      }
      id
    }
  }
`
```

The *PhoneForm* component responsible for the change is straightforward. The form has fields for the person's name and new phone number, and calls the `changeNumber` function. The function is done using the `useMutation` hook.
```
import { useState } from 'react'
import { useMutation } from '@apollo/client'

import { EDIT_NUMBER } from '../queries'

const PhoneForm = () => {
  const [name, setName] = useState('')
  const [phone, setPhone] = useState('')

  const [ changeNumber ] = useMutation(EDIT_NUMBER)
  const submit = (event) => {
    event.preventDefault()

    changeNumber({ variables: { name, phone } })
    setName('')
    setPhone('')
  }

  return (
    <div>
      <h2>change number</h2>

      <form onSubmit={submit}>
        <div>
          name <input
            value={name}
            onChange={({ target }) => setName(target.value)}
          />
        </div>
        <div>
          phone <input
            value={phone}
            onChange={({ target }) => setPhone(target.value)}
          />
        </div>
        <button type='submit'>change number</button>
      </form>
    </div>
  )
}

export default PhoneForm
```

Surprisingly, when a person's number is changed, the new number automatically appears on the list of persons rendered by the *Persons* component. This happens because each person has an identifying field of type *ID*, so the person's details saved to the cache update automatically when they are changed with the mutation.

Our application still has one small flaw. If we try to change the phone number for a name which does not exist, nothing seems to happen. This happens because if a person with the given name cannot be found, the mutation response is *null*.

For GraphQL, this is not an error, so registering an `onError` error handler is not useful.

We can use the `result` field returned by the `useMutation` hook as its second parameter to generate an error message:
```
const PhoneForm = ({ setError }) => {
  const [name, setName] = useState('')
  const [phone, setPhone] = useState('')

  const [ changeNumber, result ] = useMutation(EDIT_NUMBER)
  const submit = (event) => {
    // ...
  }

  useEffect(() => {
    if (result.data && result.data.editNumber === null) {
      setError('person not found')
    }
  }, [result.data])

  // ...
}
```

If a person cannot be found, or the `result.data.editNumber` is `null`, the component uses the callback function it received as props to set a suitable error message. We want to set the error message only when the result of the mutation `result.data` changes, so we use the useEffect hook to control setting the error message.

Using useEffect causes an ESLint warning, saying that useEffect has a missing dependency setError.

It would be best to ignore the ESLint rule on the line:
```
useEffect(() => {
  if (result.data && !result.data.editNumber) {
    setError('name not found')
  }
}, [result.data])  // eslint-disable-line 
```

We could try to get rid of the warning by adding the `setError` function to useEffect's second parameter array. However, this solution does not work if the `notify` function is not wrapped to a *useCallback* function. If it's not, this results in an endless loop. When the *App* component is rerendered after a notification is removed, a *new version* of `notify` gets created, which causes the effect function to be executed, which causes a new notification, and so on.

## Apollo Client and the application's state

In our example, management of the application's state has mostly become the responsibility of Apollo Client. This is a quite typical solution for GraphQL applications. Our example uses the state of the React components only to manage the state of a form and to show error notifications. As a result, it could be that there are no justifiable reasons to use Redux to manage application state when using GraphQL.

When necessary, Apollo enables saving the application's local state to Apollo cache.
