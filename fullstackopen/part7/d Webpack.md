# Webpack

## Bundling

We have implemented our applications by dividing our code into separate modules that have been *imported* to places that require them. Even though ES6 modules are defined in the ECMAScript standard, the older browsers do not know how to handle code that is divided into modules.

For this reason, code that is divided into modules must be *bundled* for browsers, meaning that all of the source code files are transformed into a single file that contains all of the application code. Previously, in part 3, we performed the bundling of our application with the `npm run build` command. Under the hood, the npm script bundles the source code using webpack, which produces the following collection of files in the *build* directory:
```
.
├── asset-manifest.json
├── favicon.ico
├── index.html
├── logo192.png
├── logo512.png
├── manifest.json
├── robots.txt
└── static
    ├── css
    │   ├── main.1becb9f2.css
    │   └── main.1becb9f2.css.map
    └── js
        ├── main.88d3369d.js
        ├── main.88d3369d.js.LICENSE.txt
        └── main.88d3369d.js.map
```

The *index.html* file located at the root of the build directory is the "main file" of the application which loads the bundled JavaScript file with a *script* tag:
```
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8"/>
    <title>React App</title>
    <script defer="defer" src="/static/js/main.88d3369d.js"></script> 
    <link href="/static/css/main.1becb9f2.css" rel="stylesheet">
  </head>
    <div id="root"></div>
  </body>
</html>
```

As we can see from the example application created with create-react-app, the build script also bundles the application's CSS files into a single */static/css/main.1becb9f2.css* file.

In practice, bundling is done so that we define an entry point for the application, which typically is the *index.js* file. When webpack bundles the code, it includes all of the code that the entry point imports, the code that its imports import, and so on.

Since part of the imported files are packages like React, Redux, and Axios, the bundled JavaScript file will also contain the contents of each of these libraries.

Note: The old way of dividing the application code into multiple files was based on the fact that *index.html* loaded all of the separate JavaScript files of the application with the help of script tags. This resulted in decreased performance, since the loading of each separate file results in some overhead. Because of this, the new preferred method is to bundle the code into a single file.

Next, we will create a suitable webpack configuration for a React application by hand from scratch.

Let's create a new directory for the project with the following subdirectories (*build* and *src*) and files:
```
├── build
├── package.json
├── src
│   └── index.js
└── webpack.config.js
```

The contents of *package.json* can be, for example:
```
{
  "name": "webpack-part7",
  "version": "0.0.1",
  "description": "practising webpack",
  "scripts": {},
  "license": "MIT"
}
```

Let's install webpack with the command:
```
npm install --save-dev webpack webpack-cli
```

We define the functionality of webpack in the *webpack.config.js* file, which we initialize with the following content:
```
const path = require('path')

const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  }
}
module.exports = config
```

We will then define a new npm script called *build* that will execute the bundling with webpack:
```
// ...
"scripts": {
  "build": "webpack --mode=development"
},
// ...
```

When we execute the `npm run build` command, our application code will be bundled by webpack. The operation will produce a new *main.js* file that is added under the *build* directory.

## Configuration file

Let's take a closer look at the contents of our current *webpack.config.js* file:
```
const path = require('path')

const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  }
}

module.exports = config
```

The configuration file has been written in JavaScript and the configuration object is exported using Node's module syntax.

The entry property of the configuration object specifies the file that will serve as the entry point for bundling the application.

The output property defines the location where the bundled code will be stored. The target directory must be defined as an *absolute path*, which is easy to create with the `path.resolve` method. We also use `__dirname`, which is a global variable in Node that stores the path to the current directory.

## Bundling React

Next, let's transform our application into a minimal React application. Let's install the required libraries:
```
npm install react react-dom
```

And let's turn our application into a React application by adding the familiar definitions in the *index.js* file:
```
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

We will also make the following changes to the *App.js* file:
```
import React from 'react' // we need this now also in component files

const App = () => {
  return (
    <div>
      hello webpack
    </div>
  )
}

export default App
```

We still need the *build/index.html* file that will serve as the "main page" of our application that will load our bundled JavaScript code with a *script* tag:
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="text/javascript" src="./main.js"></script>
  </body>
</html>
```

When we bundle our application, we run into the following problem:
Module parse failed: Unexpected token
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file.

## Loaders

The error message from webpack states that we may need an appropriate *loader* to bundle the *App.js* file correctly. By default, webpack only knows how to deal with plain JavaScript, and does not know how to handle the JSX we use in React.

We can use *loaders* to inform webpack of the files that need to be processed before they are bundled.

Let's configure a loader to our application that transforms the JSX code into regular JavaScript:
```
const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-react'],
        },
      },
    ],  
  },
}
```

Loaders are defined under the *module* property in the *rules* array.

The definition of a single loader consists of three parts. The *test* property specifies that the loader is for files that have names ending with *.js*. The *loader* property specifies that the processing for those files will be done with *babel-loader*. The *options* property is used for specifying parameters for the loader, which configure its functionality.

Let's install the loader and its required packages as a *development dependency*:
```
npm install @babel/core babel-loader @babel/preset-react --save-dev
```

Bundling the application will now succeed.

If we make some changes to the *App* component and take a look at the bundled code, we notice that the bundled version of the component looks like this:
```
const App = () =>
  react__WEBPACK_IMPORTED_MODULE_0___default.a.createElement(
    'div',
    null,
    'hello webpack'
  )
```

As we can see from the example above, the React elements that were written in JSX are now created with regular JavaScript by using React's `createElement` function.

It's worth noting that if the bundled application's source code uses *async/await*, the browser will not render anything on some browsers. We have to install two more missing dependencies, *core-js* and *regenerator-runtime*:
```
npm install core-js regenerator-runtime
```

We also need to import those dependencies at the top of the *index.js* file:
```
import 'core-js/stable/index.js'
import 'regenerator-runtime/runtime.js'
```

## Transpilers

The process of transforming code from one form of JavaScript to another is called transpiling. The general definition of the term is to compile source code by transforming it from one language to another.

By using the configuration from the previous section, we are *transpiling* the code containing JSX into regular JavaScript with the help of *babel*.

As mentioned before, most browsers do not support the latest features in ES6 and ES7, so code is usually transpiled to a version of JavaScript that implements the ES5 standard.

The transpilation process that is executed by Babel is defined with *plugins*. In practice, most developers use ready-made presets that are groups of pre-configured plugins.

Currently, we are using the @babel/preset-react preset. Let's add the @babel/preset-env plugin that contains everything needed to take code using all of the latest features and transpile it to code compatible with the ES5 standard:
```
{
  test: /\.js$/,
  loader: 'babel-loader',
  options: {
    presets: ['@babel/preset-env', '@babel/preset-react']  }
}
```

Let's install the preset with the command:
```
npm install @babel/preset-env --save-dev
```

## CSS

Let's add CSS to our application. Let's create a new *src/index.css* file:
```
.container {
  margin: 10px;
  background-color: #dee8e4;
}
```

Then let's use the style in the *App* component:
```
const App = () => {
  return (
    <div className="container">
      hello webpack
    </div>
  )
}
```

And import the style in the *index.js* file:
```
import './index.css'
```

This will cause the transpilation process to break. When using CSS, we have to use css and style loaders:
```
{
  rules: [
    {
      test: /\.js$/,
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-react', '@babel/preset-env'],
      },
    },
    {
      test: /\.css$/,
      use: ['style-loader', 'css-loader'],
    },
  ];
}
```

The job of the css loader is to load the *CSS* files, and the job of the style loader is to generate and inject a *style* element that contains all of the styles of the application.

With this configuration, the CSS definitions are included in the *main.js* file of the application. For this reason, there is no need to separately import the *CSS* styles in the main *index.js* file of the application.

If needed, the application's CSS can also be generated into its own separate file by using the *mini-css-extract-plugin*.

When we install the loaders:
```
npm install style-loader css-loader --save-dev
```

The bundling will succeed again, and the application gets new styles.

## Webpack-dev-server

