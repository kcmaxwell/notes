# GraphQL-server

REST, familiar to us from previous parts of the course, has long been the most prevalent way to implement the interfaces servers offer for browsers, and in general, the integration between different applications on the web.

In recent years, GraphQL, developed by Facebook, has become popular for communication between web applications and servers.

The GraphQL philosophy is very different from REST. REST is *resource-based*. Every resource, for example a *user*, has its own address which identifies it, for example */users/10*. All operations done to the resource are done with HTTP requests to its URL. The action depends on the HTTP method used.

The resource-basedness of REST works well in most situations. However, it can be a bit awkward sometimes.

Let's consider the following example: our bloglist application contains some kind of social media functionality, and we would like to show a list of all the blogs that were added by users who have commented on any of the blogs we follow.

If the server implemented a REST API, we would probably have to do multiple HTTP requests from the browser before we had all the data we wanted. The requests would also return a lot of unnecessary data, and the code on the browser would probably be quite complicated.

If this was an often-used functionality, there could be a REST endpoint for it. If there were a lot of these kinds of scenarios, however, it would become very laborious to implement REST endpoints for all of them.

A GraphQL server is well-suited for these kinds of situations.

The main principle of GraphQL is that the code on the browser forms a *query* describing the data wanted, and sends it to the API with an HTTP POST request. Unlike REST, all GraphQL queries are sent to the same address, and their type is POST.

The data described in the above scenario could be fetched with (roughly) the following query:
```
query FetchBlogsQuery {
  user(username: "mluukkai") {
    followedUsers {
      blogs {
        comments {
          user {
            blogs {
              title
            }
          }
        }
      }
    }
  }
}
```

The content of the `FetchBlogsQuery` can be roughly interpreted as: find a user named `"mluukkai"` and for each of his `followedUsers`, find all of their `blogs`, and for each blog, all its `comments`, and for each `user` who wrote each comment, find their `blogs` and return the `title` of each of them.

The server's response would be about the following JSON object:
```
{
  "data": {
    "followedUsers": [
      {
        "blogs": [
          {
            "comments": [
              {
                "user": {
                  "blogs": [
                    {
                      "title": "Goto considered harmful"
                    },
                    {
                      "title": "End to End Testing with Cypress is most enjoyable"
                    },
                    {
                      "title": "Navigating your transition to GraphQL"
                    },
                    {
                      "title": "From REST to GraphQL"
                    }
                  ]
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```

The application logic stays simple, and the code on the browser gets exactly the data it needs with a single query.

## Schemas and queries

We will get to know the basics of GraphQL by implementing a GraphQL version of the phonebook application from parts 2 and 3.

In the heart of all GraphQL applications is a schema, which describes the data sent between the client and the server. The initial schema for our phonebook is as follows:
```
type Person {
  name: String!
  phone: String
  street: String!
  city: String!
  id: ID! 
}

type Query {
  personCount: Int!
  allPersons: [Person!]!
  findPerson(name: String!): Person
}
```

The schema describes two types. The first type, *Person*, determines that persons have five fields. Four of the fields are type *String*, which is one of the scalar types of GraphQL. All of the String fiels, except *phone*, must be given a value. This is marked by the exclamation mark on the schema. The type of the field *id* is *ID*. *ID* fields are strings, but GraphQL ensures they are unique.

The second type is a Query. Practically every GraphQL schema describes a Query, which tells what kind of queries can be made to the API.

The phonebook describes three different queries. `personCount` returns an integer, `allPersons` returns a list of *Person* objects, and *findPerson* is given a string parameter and it returns a *Person* object.

Again, exclamation marks are used to mark which return values and parameters are *Non-Null*. `personCount` will, for sure, return an integer. The query `findPerson` must be given a string as a parameter. The query returns a *Person*-object or *null*. `allPersons` returns a list of *Person* objects, and the list does not contain any *null* values.

So the schema describes what queries the client can send to the server, what kind of parameters the queries can have, and what kind of data the queries return.

The simplest of the queries, `personCount`, looks as follows:
```
query {
  personCount
}
```

Assuming our application has saved the information of three people, the response would look like this:
```
{
  "data": {
    "personCount": 3
  }
}
```

The query fetching the information of all of the people, `allPersons`, is a bit more complicated. Because the query returns a list of *Person* objects, the query must describe *which fields* of the objects the query returns:
```
query {
  allPersons {
    name
    phone
  }
}
```

The response could look like this:
```
{
  "data": {
    "allPersons": [
      {
        "name": "Arto Hellas",
        "phone": "040-123543"
      },
      {
        "name": "Matti Luukkainen",
        "phone": "040-432342"
      },
      {
        "name": "Venla Ruuska",
        "phone": null
      }
    ]
  }
}
```

A query can be made to return any field described in the schema.

The last example shows a query which requires a parameter, and returns the details of one person:
```
query {
  findPerson(name: "Arto Hellas") {
    phone 
    city 
    street
    id
  }
}
```

So first, the parameter is described in round brackets, and then the fields of the return value object are listed in curly brackets.

Ther response is like this:
```
{
  "data": {
    "findPerson": {
      "phone": "040-123543",
      "city": "Espoo",
      "street": "Tapiolankatu 5 A"
      "id": "3d594650-3436-11e9-bc57-8b80ba54c431"
    }
  }
}
```

The return value was marked as nullable, so if we search for the details of an unknown person, the return value is *null*.

As you can see, there is a direct link between a GraphQL query and the returned JSON object. One can think that the query describes what kind of data it wants as a response. The difference to REST queries is stark. With REST, the URL and the type of the request have nothing to do with the form of the returned data.

GraphQL query describes only the data moving between a server and the client. On the server, the data can be organized and saved any way we like.

Despite its name, GraphQL does not actually have anything to do with databases. It does not care how the data is saved. The data a GraphQL API uses can be saved into a relational database, document database, or to other servers which a GraphQL server can access with, for example, REST.

## Apollo Server

Let's implement a GraphQL server with today's leading library: Apollo Server.

Create a new npm project with `npm init` and install the required dependencies:
```
npm install apollo-server@3.10.1 graphql
```

**Note** at the time of writing, the code in this part is not fully compatible with the new version of Apollo Server, so we have to install version 3.10.1.

Also create a `index.js` in your project's root directory.

The initial code is as follows:
```
const { ApolloServer, gql } = require('@apollo/server')

let persons = [
  {
    name: "Arto Hellas",
    phone: "040-123543",
    street: "Tapiolankatu 5 A",
    city: "Espoo",
    id: "3d594650-3436-11e9-bc57-8b80ba54c431"
  },
  {
    name: "Matti Luukkainen",
    phone: "040-432342",
    street: "Malminkaari 10 A",
    city: "Helsinki",
    id: '3d599470-3436-11e9-bc57-8b80ba54c431'
  },
  {
    name: "Venla Ruuska",
    street: "Nallemäentie 22 C",
    city: "Helsinki",
    id: '3d599471-3436-11e9-bc57-8b80ba54c431'
  },
]

const typeDefs = gql`
  type Person {
    name: String!
    phone: String
    street: String!
    city: String! 
    id: ID!
  }

  type Query {
    personCount: Int!
    allPersons: [Person!]!
    findPerson(name: String!): Person
  }
`

const resolvers = {
  Query: {
    personCount: () => persons.length,
    allPersons: () => persons,
    findPerson: (root, args) =>
      persons.find(p => p.name === args.name)
  }
}

const server = new ApolloServer({
  typeDefs,
  resolvers,
})

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`)
})
```

The heart of the code is an `ApolloServer`, which is given two parameters. The first parameter, `typeDefs`, contains the GraphQL schema. The second parameter is an object, which contains the resolvers of the server. These are the code which defines *how* GraphQL queries are responded to.

The code of the resolvers is the following:
```
const resolvers = {
  Query: {
    personCount: () => persons.length,
    allPersons: () => persons,
    findPerson: (root, args) =>
      persons.find(p => p.name === args.name)
  }
}
```

As you can see, the resolvers correspond to the queries described in the schema. There is a field under *Query* for every query described in the schema.

Start the server by running `node index.js` in the terminal.

## Apollo Studio Explorer

When Apollo server is run in development mode, the page http://localhost:4000 has a button *Query your server* that takes us to Apollo Studio Explorer. This is very useful for a developer, and can be used to make queries to the server.

At the left side, Explorer shows the API-documentation that it has automatically generated based on the schema.

## Parameters of a resolver

