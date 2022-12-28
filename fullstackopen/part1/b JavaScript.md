# JavaScript

The name of the JavaScript standard is ECMAScript. Currently, the latest version is ECMAScript 2021, otherwise known as ES12.

Browsers do not yet support all of JavaScript's newest features, so code run in browsers has been *transpiled* from a newer version of JavaScript to an older, more compatible version.

The most popular way to do transpiling is by using Babel. Transpilation is automatically configured in React applications created with create-react-app.

Node.js is a JavaScript runtime environment based on Google's Chrome V8 JavaScript engine, and works practically anywhere.

You can run JavaScript code in .js files using the command `node name_of_file.js`.
You can also write JavaScript code in the Node.js console, which you open using the command `node`.

## Variables

You can define variables using const, let, and var.

const does not actually define a variable, but a constant whose value cannot be changed.
let defines a normal variable.

var used to be the only way to define variables, until let and const were added in ES6.
var has function scope, while let has block scope.

## Arrays

An array can be defined like so: `const t = [0, 1, 3]`.
You can still edit the values in the array if you use const. Because the array is an object, the variable will always point to the same object, but the contents are allowed to change.
You can access an index using `t[0]`, and get the length using `t.length`.

One way of iterating through the items of the array is using forEach:
```
t.forEach(value => {
    console.log(value)
});
```

forEach calls the function for each of the items in the array, always passing the individual item as an argument. The function as the argument of forEach can also receive other arguments.
```
forEach((element) => { ... })
forEach((element, index) => { ... })
forEach((element, index, array) => { ... })
```

When using React, techniques from functional programming are often used. One characteristic of the functional programming paradigm is the use of immutable data structures. 
For example, in React code, it is preferable to use the method concat instead of push for arrays. This will create a new array containing the old array and the new item, instead of modifying the old array:
`const t2 = t.concat(5)`

The map method creates a new array, with the function given as a parameter creating the items in the new array. Map can also transform the array into something completely different. For example: 
```
const t = [1, 2, 3]
const m1 = t.map(value => value * 2)
const m2 = t.map(value => '<li>' + value + '</li>')
```

You can assign the elements in an array to individual elements using a destructuring assignment:
```
const t = [1, 2, 3, 4, 5]
const [first, second, ...rest] = t
console.log(first, second) // prints 1, 2
console.log(rest) // prints [3, 4, 5]
```
The variables `first` and `second` will receive the first two integers of the array as their values, and the remaining integers are collected into an array of their own, and assigned to the variable `rest`.

## Objects
One way to define objects is using object literals, by listing its properties in braces:
```
const object1 = {
    name: 'Kristopher Maxwell',
    age: 30,
    education: 'BSc',
}
```
The values of the properties can be of any type, like integers, strings, arrays, objects, etc.

The properties of an object are referenced using dot notation, or by using brackets:
```
console.log(object1.name) // prints Kristopher Maxwell
const fieldName = 'age'
console.log(object1[fieldName]) // prints 30
```

You can add properties to an object using dot notation or brackets:
```
object1.address = 'Calgary'
object1['secret number'] = 13
```
The latter of trhe additions has to be done using brackets, because *secret number* is not a valid property name when using dot notation, because of the space character.

Objects in JavaScript can also have methods.
Objects can also be defined using constructor functions.

## Functions

To define and use an arrow function:
```
const sum = (p1, p2) => {
    console.log(p1)
    console.log(p2)
    return p1 + p2
}

const result = sum(1, 5)
console.log(result) // prints 6
```

If there is only a single parameter, you can exclude the parentheses:
```
const square = p => {
    console.log(p)
    return p * p
}
```

If the function only contains a single expression, the braces are not needed, and the function will only return the result of its only expression:
`const square = p => p * p`

The arrow function feature was added to JavaScript in version ES6. Prior to that, the only way to define functions was by using the `function` keyword.

There are two ways to reference the function; one is by giving a name in a function declaration:
```
function product(a, b) {
    return a * b
}

const result = product(2, 6)
```

The other way is by using a function expression (using the function keyword to define a function inside an expression). In this case, there is no need to give the function a name:
```
const average = function(a, b) {
    return (a + b) / 2
}

const result = average(2, 5)
```

## Object methods and "this"

Arrow functions and functions defined using `function` vary substantially with how they behave with respect to the keyword `this`, which refers to the object itself.

We can assign methods to an object by defining properties that are functions:
```
const arto = {
  name: 'Arto Hellas',
  age: 35,
  education: 'PhD',
  greet: function() {
    console.log('hello, my name is ' + this.name)
  },
}

arto.greet()  // "hello, my name is Arto Hellas" gets printed
```

Methods can be assigned to objects even after the creation of the object:
```
const arto = {
  name: 'Arto Hellas',
  age: 35,
  education: 'PhD',
  greet: function() {
    console.log('hello, my name is ' + this.name)
  },
}

arto.growOlder = function() {  this.age += 1}
console.log(arto.age)   // 35 is printed
arto.growOlder()
console.log(arto.age)   // 36 is printed
```

Here is an example of storing a *method reference* in a variable outside of the object, and how `this` can cause problems:
```
const object = {
    name: 'Object',
    identify: function() {
        console.log('My name is ' + this.name)
    },
    doAddition: function(a, b) {
        console.log(a + b)
    },
}

object.doAddition(1, 4) // prints 5
const referenceDoAddition = object.doAddition
referenceDoAddition(1, 4) // prints 5

object.identify() // prints "My name is object"
const referenceIdentify = object.identify()
referenceIdentify() // prints "My name is undefined"
```

In the above example, the method reference for the identify function returns undefined for `this.name`. When calling the method through a reference, the method loses knowledge of what the original `this` was. JavaScript defines the value of `this` based on *how the method is called*. When calling the method through a reference, the value of `this` becomes the global object, instead of the object the function belongs to.

Antoher situation that can cause this is when we set a timeout to call the method using the setTimeout function:
`setTimeout(object.identify(), 1000)`
This will print "My name is undefined" in 1000 ms, because the JavaScript engine will call the method, and at that point, `this` refers to the global object.

One way to preserve the original `this` is by using a method called bind:
`setTimeout(object.identify.bind(object), 1000)`
Calling `object.identify.bind(object)` creates a new function where `this` is bound to point to object, independent of where and how the method is being called.

Using arrow functions, it is possible to avoid some problems of `this`. However, you should not use them as methods for objects, because then `this` does not work at all.

## Classes

There is no class mechanism in JavaScript like the ones in object-oriented programming languages. However, there are features to make "simulating" object-oriented classes possible.

The class syntax was introduced to JavaScripte in ES6, and substantially simplifies the definition of classes in JavaScript.
```
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }
  greet() {
    console.log('hello, my name is ' + this.name)
  }
}

const adam = new Person('Adam Ondra', 29)
adam.greet()

const janja = new Person('Janja Garnbret', 23)
janja.greet()
```

The classes and objects created are reminiscent of Java classes and objects. Their behavior is also similar to Java objects. At the core, they are still objects based on JavaScript's prototypal inheritance. The type of both objects is actually `Object`, since JavaScript essentially only defines the types Boolean, Null, Undefined, Number, String, Symbol, BigInt, and Object.

ES6 class syntax is used a lot in old React and in Node.js, but is not as necessary when using React Hooks.

## JavaScript materials

For further reading of JavaScript:

Mozilla's JavaScript Guide: https://developer.mozilla.org/en-US/docs/Web/JavaScript

A re-introduction to JavaScript: https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript

A free book series You-Dont-Know-JS: https://github.com/getify/You-Dont-Know-JS

Javascript.info: https://javascript.info/

Eloquent JavaScript: https://eloquentjavascript.net/
