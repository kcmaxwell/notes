# Login and updating the cache

The frontend of our application shows the phone directory fine with the updated server, but if we want to add new persons, we have to add login functionality to the frontend.

## User login

Let's add the variable `token` to the application's state. When a user is logged in, it will contain a user token. If `token` is undefined, we render the *LoginForm* component responsible for user login. The component receives an error handler and the `setToken` function as parameters:
```
const App = () => {
  const [token, setToken] = useState(null)

  // ...

  if (!token) {
    return (
      <div>
        <Notify errorMessage={errorMessage} />
        <h2>Login</h2>
        <LoginForm
          setToken={setToken}
          setError={notify}
        />
      </div>
    )
  }

  return (
    // ...
  )
}
```

Next, we define a mutation for logging in:
```
export const LOGIN = gql`
  mutation login($username: String!, $password: String!) {
    login(username: $username, password: $password)  {
      value
    }
  }
`
```

The `LoginForm` component works just like all the other components doing mutations from before:
```
import { useState, useEffect } from 'react'
import { useMutation } from '@apollo/client'
import { LOGIN } from '../queries'

const LoginForm = ({ setError, setToken }) => {
  const [username, setUsername] = useState('')
  const [password, setPassword] = useState('')

  const [ login, result ] = useMutation(LOGIN, {    onError: (error) => {
      setError(error.graphQLErrors[0].message)
    }
  })

  useEffect(() => {
    if ( result.data ) {
      const token = result.data.login.value
      setToken(token)
      localStorage.setItem('phonenumbers-user-token', token)
    }
  }, [result.data]) // eslint-disable-line

  const submit = async (event) => {
    event.preventDefault()

    login({ variables: { username, password } })
  }

  return (
    <div>
      <form onSubmit={submit}>
        <div>
          username <input
            value={username}
            onChange={({ target }) => setUsername(target.value)}
          />
        </div>
        <div>
          password <input
            type='password'
            value={password}
            onChange={({ target }) => setPassword(target.value)}
          />
        </div>
        <button type='submit'>login</button>
      </form>
    </div>
  )
}

export default LoginForm
```

We are using an effect hook to save the token's value to the state of the `App` component and the local storage after the server has responded to the mutation. Use of the effect hook is necessary to avoid an endless rendering loop.

Let's also add a button which enables a logged-in user to log out. The button's onClick handler sets the `token` state to null, removes the token from local storage, and resets the cache of the Apollo client. The last step is important, because some queries might have fetched data to cache, which only logged-in users should have access to.

We can reset the cache using the *resetStore* method of an Apollo client object. The client can be accessed with the *useApolloClient* hook:
```
const App = () => {
  const [token, setToken] = useState(null)
  const [errorMessage, setErrorMessage] = useState(null)
  const result = useQuery(ALL_PERSONS)
  const client = useApolloClient()

  if (result.loading)  {
    return <div>loading...</div>
  }

  const logout = () => {
    setToken(null)
    localStorage.clear()
    client.resetStore()
  }

  if (!token) {
    return (
      <>
        <Notify errorMessage={errorMessage} />
        <LoginForm setToken={setToken} setError={notify} />
      </>
    )
  }

  return (
    <>
      <Notify errorMessage={errorMessage} />
      <button onClick={logout}>logout</button>
      <Persons persons={result.data.allPersons} />
      <PersonForm setError={notify} />
      <PhoneForm setError={notify} />
    </>
  )
}
```

## Adding a token to a header

After the backend changes, creating new persons requires that a valid user token is sent with the request. In order to send the token, we have to change the way we define the `ApolloClient` object in *index.js* a little.
```
import { ApolloClient, InMemoryCache, ApolloProvider, createHttpLink } from '@apollo/client'import { setContext } from '@apollo/client/link/context'
const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('phonenumbers-user-token')
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : null,
    }
  }
})

const httpLink = createHttpLink({
  uri: 'http://localhost:4000',
})

const client = new ApolloClient({
  cache: new InMemoryCache(),
  link: authLink.concat(httpLink)
})
```

The field `uri` that was previously used when creating the `client` object has been replaced by the field `link`, which defines in a more complicated case how Apollo is connected to the server. The server url is now wrapped using the function *createHttpLink* into a suitable httpLink object. The link is modified by the context defined by the authLink object so that a possible token in localStorage is set to header *authorization* for each request to the server.

Creating new persons and changing numbers works again. There is however one remaining problem. If we try to add a person without a phone number, it is not possible.

Validation fails, because frontend sends an empty string as the value of `phone`.

Let's change the function creating new persons so that it sets `phone` to `undefined` if user has not given a value.
```
const PersonForm = ({ setError }) => {
  // ...
  const submit = async (event) => {
    event.preventDefault()
    createPerson({
      variables: { 
        name, street, city,
        phone: phone.length > 0 ? phone : undefined
      }
    })

  // ...
  }

  // ...
}
```

## Updating cache, revisited

We have to update the cache of the Apollo client on creating new persons. We can update it using the mutation's `refetchQueries` option to define that the `ALL_PERSONS` query is done again.
```
const PersonForm = ({ setError }) => {
  // ...

  const [ createPerson ] = useMutation(CREATE_PERSON, {
    refetchQueries: [  {query: ALL_PERSONS} ],
    onError: (error) => {
      const errors = error.graphQLErrors[0].extensions.error.errors
      const messages = Object.values(errors).map(e => e.message).join('\n')
      setError(messages)
    }
  })
```

This approach is pretty good, the drawback being that the query is always rerun with any updates.

It is possible to optimize the solution by handling updating the cache ourselves. This is done by defining a suitable update callback for the mutation, which Apollo runs after the mutation:
```
const PersonForm = ({ setError }) => {
  // ...

  const [ createPerson ] = useMutation(CREATE_PERSON, {
    onError: (error) => {
      setError(error.graphQLErrors[0].message)
    },
    update: (cache, response) => {
      cache.updateQuery({ query: ALL_PERSONS }, ({ allPersons }) => {
        return {
          allPersons: allPersons.concat(response.data.addPerson),
        }
      })
    },
  })
 
  // ..
}
```

The callback function is given a reference to the cache and the data returned by the mutation as parameters. For example, in our case, this would be the created person.

Using the function updateQuery, the code updates the query `ALL_PERSONS` in cache by adding the new persons to the cached data.

In some situations, the only sensible way to keep the cache up to date is using the `update` callback.

When necessary, it is possible to disable cache for the whole application or single queries by setting the field managing the use of cache, fetchPolicy as `no-cache`.

Be diligent with the cache. Old data in cache can cause hard-to-find bugs.
