# React with types

Before we start delving into how you can use TypeScript with React, we should first have a look at what we want to achieve. When everything works as it should, TypeScript will help us catch the following errors:
- Trying to pass an extra/unwanted prop to a component
- Forgetting to pass a required prop to a component
- Passing a prop with the wrong type to a component

If we make any of these errors, TypeScript can help us catch them in our editor right away. If we didn't use TypeScript, we would have to catch these errors later during testing. We might be forced to do some tedious debugging to find the cause of the errors.

## Create React App with TypeScript

We can use create-react-app to create a TypeScript app by adding a *template* argument to the initialization script. So in order to create a TypeScript Create React App, run the following command:
```
npx create-react-app my-app --template typescript
```

After running the command, you should have a complete basic React app that uses TypeScript. You can start the app by running `npm start` in the application's root.

If you take a look at the files and folders, you'll notice that the app is not that different from one using pure JavaScript. The only differences are that the *.js* and *.jsx* files are now *.ts* and *.tsx* files, they contain some type annotations, and the root directory contains a *tsconfig.json* file.

Now, let's take a look at the *tsconfig.json* file that has been created for us:
```
{
  "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": [
    "src"
  ]
}
```

Notice `compilerOptions` now has the key *lib* that includes "type definitions for things found in browser environments (like `document`)".

Everything else should be more or less fine, except that at the moment, the configuration allows compiling JavaScript files because `allowJs` is set to `true`. That would be fine if you need to mix TypeScript and JavaScript (e.g. if you are in the process of transforming a JavaScript project into TypeScript or something like that), but we want to create a pure TypeScript app, so let's change that configuration to `false`.

In our previous project, we used ESLint to help us enforce a coding style, and we'll do the same with this app. We do not need to install any dependencies, since create-react-app has taken care of that already.

We configure ESLint in *.eslintrc* with the following settings:
```
{
  "env": {
    "browser": true,
    "es6": true,
    "jest": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "plugins": ["react", "@typescript-eslint"],
  "settings": {
    "react": {
      "pragma": "React",
      "version": "detect"
    }
  },
  "rules": {
    "@typescript-eslint/explicit-function-return-type": 0,
    "@typescript-eslint/explicit-module-boundary-types": 0,
    "react/react-in-jsx-scope": 0
  }
}
```

Since the return type of most React components is generally either `JSX.Element` or `null`, we have loosened the default linting rules a bit by disabling the rules *explicit-function-return-type* and *explicit module-boundary-types*. Now we don't need to explicitly state our function return types everywhere. We will also disable *react/react-in-jsx-scope* since importing React is no longer needed in every file.

Next, we need to get our linting script to parse *\*.tsx* files, which are the TypeScript equivalent of React's JSX files. We can do that by altering our lint command in *.package.json* to the following:
```
{
  // ...
    "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "lint": "eslint './src/**/*.{ts,tsx}'"
  },
  // ...
}
```

If you are using Windows, you may need to use double quotes for the linting path:
```
"lint": "eslint \"./src/**/*.{ts,tsx}\""
```

## React components with TypeScript

Let us consider the following JavaScript React example:
```
import ReactDOM from 'react-dom/client'
import PropTypes from "prop-types";

const Welcome = props => {
  return <h1>Hello, {props.name}</h1>;
};

Welcome.propTypes = {
  name: PropTypes.string
};

ReactDOM.createRoot(document.getElementById('root')).render(
  <Welcome name="Sarah" />
)
```

In this example, we have a component called `Welcome` to which we pass a `name` as a prop. It then renders the name to the screen. We know that the `name` should be a string, and we use the prop-types package to receive hints about the desired types of a component's props and warnings about invalid prop types.

With TypeScript, we don't need the prop-types package anymore. We can define the types with the help of TypeScript just like we define types for a regular function, as React components are nothing but mere functions. We will use an interface for the parameter types (i.e. props) and `JSX.Element` as the return type for any React component:
```
import ReactDOM from 'react-dom/client'

interface WelcomeProps {
  name: string;
}

const Welcome = (props: WelcomeProps): JSX.Element => {
  return <h1>Hello, {props.name}</h1>;
};

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <Welcome name="Sarah" />
)
```

We defined a new type, `WelcomeProps`, and passed it to the function's parameter types. You could write the same thing using a more verbose syntax:
```
const Welcome = ({ name }: { name: string }): JSX.Element => (
  <h1>Hello, {name}</h1>
);
```

There is actually no need to define the return type of a React component since the TypeScript compiler infers the type automatically, and we can just write:
```
interface WelcomeProps {
  name: string;
}

const Welcome = (props: WelcomeProps)  => {
    return <h1>Hello, {props.name}</h1>;
};

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <Welcome name="Sarah" />
)
```

We used a type assertion for the return value of the function `document.getElementById`. We need to do this since the `ReactDOM.createRoo` takes an HTMLElement as a parameter, but the return value of function `document.getElementById` has the following type: `HTMLElement | null`, since if the function does not find the searched element, it will return null.

In this case, the assertion is OK since we are sure that the file *index.html* indeed has this particular id and the function is always returning an HTMLElement.

## Deeper type usage

In the previous exercise, 9.14, we had three parts of a course, and all parts had the same attributes `name` and `exerciseCount`. But what if we needed additional attributes for the parts where all parts do not have the same attributes? How would this look, codewise? Let's consider the following example:
```
const courseParts = [
  {
    name: "Fundamentals",
    exerciseCount: 10,
    description: "This is an awesome course part"
  },
  {
    name: "Using props to pass data",
    exerciseCount: 7,
    groupProjectCount: 3
  },
  {
    name: "Basics of type Narrowing",
    exerciseCount: 7,
    description: "How to go from unknown to string"
  },
  {
    name: "Deeper type usage",
    exerciseCount: 14,
    description: "Confusing description",
    backgroundMaterial: "https://type-level-typescript.com/template-literal-types"
  },
];
```

In the above example, we have added some additional attributes to each course part. Each part has the `name` and `exerciseCount` attributes, but the first, the third, and fourth also have an attribute called `description`, and the second and fourth parts also have some distinct additional attributes.

Let's imagine that our application just keeps on growing, and we need to pass the different course parts around in our code. On top of that, there are also additional attributes and course parts added to the mix. How can we know that our code is capable of handling all the different types of data correctly, and we are not for example forgetting to render a new course part on some page? This is where TypeScript comes in handy!

Let's start by defining types for our different course parts. We notice that the first and third have the same set of attributes. The second and fourth are a bit different, so we have three different kinds of course part elements.

So let us define a type for each of the different kinds of course parts:
```
interface CoursePartBasic {
  name: string;
  exerciseCount: number;
  description: string;
  kind: "basic"
}

interface CoursePartGroup {
  name: string;
  exerciseCount: number;
  groupProjectCount: number;
  kind: "group"
}

interface CoursePartBackground {
  name: string;
  exerciseCount: number;
  description: string;
  backgroundMaterial: string;
  kind: "background"
}
```

Besides the attributes that are found in the various course parts, we have now introduced an additional attribute called *kind* that has a literal type, it is a "hard coded" string, distinct for each course part.

Next, we will create a type union of all these types. We can then use it to define a type for our array, which should accept any of these course part types:
```
type CoursePart = CoursePartBasic | CoursePartGroup | CoursePartBackground;
```

Now we can set the type for our `courseParts` variable:
```
const App = () => {
  const courseName = "Half Stack application development";
  const courseParts: CoursePart[] = [
    {
      name: "Fundamentals",
      exerciseCount: 10,
      description: "This is an awesome course part",
      kind: "basic"
    },
    {
      name: "Using props to pass data",
      exerciseCount: 7,
      groupProjectCount: 3,
      kind: "group"
    },
    {
      name: "Basics of type Narrowing",
      exerciseCount: 7,
      description: "How to go from unknown to string",
      kind: "basic"
    },
    {
      name: "Deeper type usage",
      exerciseCount: 14,
      description: "Confusing description",
      backroundMaterial: "https://type-level-typescript.com/template-literal-types",
      kind: "background"
    },
  ]

  // ...
}
```

Note that we have now added the attribute `kind` with a proper value to each element of the array.

Our editor will automatically warn us if we use the wrong type for an attribute, use an extra attribute, or forget to set an expected attribute. If we, e.g. try to add the following to the array:
```
{
  name: "TypeScript in frontend",
  exerciseCount: 10,
  kind: "basic",
},
```

We will immediately see an error in the editor. Since our new entry has the attribute `kind` with value `"basic"`, TypeScript knows that the entry does not only have the type `CoursePart`, but it is actually meant to be a `CoursePartBasic`. So here, the attribute `kind` "narrows" the type of the entry from a more general to a more specific type that has a certain set of attributes. We shall soon see this style of type narrowing in action in the code.

There is still a lot of duplication in our types, and we want to avoid that. We start by identifying the attributes all course parts have in common, and define a base type that contains them. Then we will extend that base type to create our kind-specific types:
```
interface CoursePartBase {
  name: string;
  exerciseCount: number;
}

interface CoursePartBasic extends CoursePartBase {
  description: string;
  kind: "basic"
}

interface CoursePartGroup extends CoursePartBase {
  groupProjectCount: number;
  kind: "group"
}

interface CoursePartBackround extends CoursePartBase {
  description: string;
  backroundMaterial: string;
  kind: "background"
}

type CoursePart = CoursePartBasic | CoursePartGroup | CoursePartBackround;
```

## More type narrowing

How should we now use these types in our components?

If we try to access the objects in the array `courseParts: CoursePart[]`, we notice that it is possible to only access the attributes that are common to all the types in the union. The TypeScript documentation says: *TypeScript will only allow an operation (or attribute access) if it is valid for every member of the union.*

The documentation also mentions the following: *The solution is to narrow the union with code... Narrowing occurs when TypeScript can deduce a more specific type for a value based on the structure of the code.*

So once again, we need to use type narrowing.

One handy way to narrow these kinds of types in TypeScript is to use `switch case` expressions. Once TypeScript has inferred that a variable is of union type and that each type in the union contains a certain literal attribute (in our case `kind`), we can use that as a type indentifier. We can then build a switch case around that attribute and TypeScript will know which attributes are available within each case block.

The specific technique of type narrowing where a union type is narrowed based on literal attribute values is called discriminated union.

Note that the narrowing can also be done with an `if` clause.

What about adding new types? If we were to add a new course part, wouldn't it be nice to know if we had already implemented handling that type in our code? In a switch case, a new type would go to the `default` block. Sometimes, this is wholly acceptable. For instance, if you wanted to handle only specific (but not all) cases of a type union, having a default is fine. Nonetheless, it is recommended to handle all variations separately in most cases.

With TypeScript, we can use a method called exhaustive type checking. Its basic principle is that if we encounter an unexpected value, we call a function that accepts a value with the type `never` and also has the return type `never`.

A straightforward version of the function could look like this:
```
/**
 * Helper function for exhaustive type checking
 */
const assertNever = (value: never): never => {
  throw new Error(
    `Unhandled discriminated union member: ${JSON.stringify(value)}`
  );
};
```

If we now were to replace the contents of our `default` block to:
```
default:
  return assertNever(part);
```
and remove the case that handles the type `CoursePartBackground`, we would see the following error: `Argument of type 'CoursePartBackground' is not assignable to parameter of type 'never'.` This tells us that we are using a variable somewhere where it should never be used. This tells us that something needs to be fixed.
