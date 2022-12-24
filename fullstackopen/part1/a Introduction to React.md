# Introduction to React

## Component
When using React, all content that needs to be rendered is usually defined as React components.

To render HTML, we usually wrap it in a function, like so:
```
const App = () => (
  <div>
    <p>Hello world</p>
  </div>
)
```

You can render also render dynamic content inside of a component, for example:
```
const App = () => {
  const now = new Date()
  const a = 10
  const b = 20

  return (
    <div>
      <p>Hello world, it is {now.toString()}</p>
      <p>
        {a} plus {b} is {a + b}
      </p>
    </div>
  )
}
```

Any JavaScript code within the curly braces is evaluated and the result is embedded into the defined place in the HTML produced by the component.

## JSX
The layout of React components are mostly written using JSX, not HTML.
JSX returned by React components is compiled into JavaScript.
Compilation is handled by Babel, and projects created with `create-react-app` are configured to compile automatically.

In practice, JSX is very similar to HTML, with the distinction being that in JSX, you can embed dynamic content with JavaScript within curly braces.

JSX is "XML-like", meaning every tag needs to be closed, even empty elements that can be left alone in HTML.
For example, `<br>` works in HTML, but would need to be `<br />` in JSX.

## Multiple Components
A core philosophy of React is composing applications from many specialised reusable components.

Another strong convention is the idea of a *root component* called *App* at the top of the component tree of the application.
There are some situations where *App* is not the root, but will be wrapped within an appropriate utility component.

## props
You can pass data to components using props.
For example:
```
const Hello = (props) => {  
  return (
    <div>
      <p>Hello {props.name}</p>    
    </div>
  )
}
```

As an argument, the parameter receives an object, which has fields corresponding to all the "props" the user of the component defines.
An example definition of props is as follows: `<Hello name='George' />`

There can be an arbitrary number of props, and their values can be hard-coded strings or the results of JavaScript expressions.
If the value of the prop is achieved using JavaScript, it must be wrapped in curly braces.

## Some notes
When developing, advance in small steps, and leave the console open to note any errors.
If there are errors, stop coding and try to solve the errors first before continuing.

Keep in mind that **React component names must be capitalized**.

Note that the content of a React component (usually) needs to contain **one root element**.
You can use a root div, or you could use a fragment, i.e. wrapping the elements with an empty element, like so:
```
const App = () => {
  const name = 'Peter'
  const age = 10

  return (
    <>
      <h1>Greetings</h1>
      <Hello name='Maya' age={26 + 10} />
      <Hello name={name} age={age} />
      <Footer />
    </>
  )
}
```
