# First steps with TypeScript

We will focus in this part on the most common issues that aries when developing Express backends or React frontends with TypeScript. In addition to language features, we will also have a strong emphasis on tooling.

## Setting things up

Install TypeScript support to your editor of choice. Visual Studio Code works natively with TypeScript.

As mentioned earlier, TypeScript code is not executable by itself. It has to be first compiled into executable JavaScript. When TypeScript is compiled into JavaScript, the code becomes subject to type erasure. This means that type annotations, interfaces, type aliases, and other type system constructs are removed and the result is pure ready-to-run JavaScript.

In a production environment, the need for compilation often means that you have to set up a "build step". During the build step, all TypeScript code is compiled into JavaScript in a separate folder, and the production environment then runs the code from that folder. In a development environment, it is often handier to make use of real-time compilation and auto-reloading so one can see the resulting changes more quickly.

Let's start writing our first TypeScript app. To keep things simple, let's start by using the npm package ts-node. It compiles and executes the specified TypeScript file immediately so that there is no need for a separate compilation step.

You can install both *ts-node* and the official *typescript* package globally by running:
```
npm install -g ts-node typescript
```

If you can't or don't want to install global packages, you can create an npm project which has the required dependencies and run your scripts in it. We will also take this approach.

We start by running the command `npm init` in an empty directory, and then install the dependencies by running:
```
npm install --save-dev ts-node typescript
```

Then, we set up *scripts* in *package.json*:
```
{
  // ..
  "scripts": {
    "ts-node": "ts-node"  
  },
  // ..
}
```

You can now use *ts-node* within this directory by running `npm run ts-node`. Note that if you are using ts-node through package.json, command-lin arguments that include short or long form options for the `npm run script` need to be prefixed with `--`. So if you want to run file.ts with ts-node and options `-s` and `--someoption`, the whole command is:
```
npm run ts-node file.ts -- -s --someoption
```

It is worth mentioning that TypeScript also provides an online playground, where you can quickly try out TypeScript code and instantly see the resulting JavaScript and possible compilation errors. You can access TypeScript's official playground here:
https://www.typescriptlang.org/play/index.html

Note that the playground might contain different tsconfig rules than your local environment, which is why you might see different warnings there compared to your local environment. The playground's tsconfig is modifiable through the config dropdown menu.

### A not about the coding style

JavaScript is quite a relaxed language, and things can often be done in multiple different ways. For example, we have named vs anonymous functions, using const and let or var, and the use of semicolons. This part of the course differs from the rest by using semicolons. It is not a TypeScript-specific pattern, but a general coding style decision taken when creating any kind of JavaScript project. Whether to use them or not is usually in the hands of the programmer, but since it is expected to adapt one's coding habits to the existing codebase, you are expected to use semicolons and adjust to the coding style in the exercises for this part. This part has some other coding style differences compared to the rest of the course as well, e.g. the directory naming conventions.

Let us add a configuration file `tsconfig.json` to the project with the following content:
```
{
  "compilerOptions":{
    "noImplicitAny": false
  }
}
```

The *tsconfig.json* file is used to define how the TypeScript compiler should interpret the code, how strictly the compiler should work, which files to watch or ignore, and much more. For now, we will only use the compiler option noImplicitAny, which does not require having types for all variables used.

Let's start by creating a simple Multiplier. It looks exactly as it would in JavaScript:
```
const multiplicator = (a, b, printText) => {
  console.log(printText,  a * b);
}

multiplicator(2, 4, 'Multiplied numbers 2 and 4, the result is:');
```

As you can see, this is still ordinary basic JavaScript with no additional TS features. It compiles and runs nicely with `npm run ts-node -- multiplier.ts`, as it would with Node.

But what happens if we end up passing the wrong *types* of arguments to the multiplicator function?
```
const multiplicator = (a, b, printText) => {
  console.log(printText,  a * b);
}

multiplicator('how about a string?', 4, 'Multiplied a string and 4, the result is:');
```

Now when we run the code, the output is: `Multiplied a string and 4, the result is: NaN`.

Wouldn't it be nice if the language itself could prevent us from ending up in situations like this? This is where we see the first benefits of TypeScript. Let's add types to the parameters and see where it takes us.

TypeScript natively supports multiple types, including `number`, `string`, and `Array`. See the comprehensive list here: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
More complex custom types can also be created.

The first two parameters of our function are the number and the string primitives, respectively:
```
const multiplicator = (a: number, b: number, printText: string) => {
  console.log(printText,  a * b);
}

multiplicator('how about a string?', 4, 'Multiplied a string and 4, the result is:');
```

When we try to run the code, we notice that it does not compile.

One of the best things about TypeScript's editor support is that you don't necessarily need to even run the code to see the issues. The VSCode plugin is so efficient that it informs you immediately when you are trying to use an incorrect type.

## Creating your first own types

Let's expand our multiplicator into a slightly more versatile calculator that also supports addition and division. The calculator should accept three arguments: two numbers and the operation, either `multiply`, `add`, or `divide`, which tells it what to do with the numbers.

In JavaScript, the code would require additional validation to make sure the last argument is indeed a string. TypeScript offers a way to define specific types for inputs, which describe exactly what kind of input is acceptable. On top of that, TypeScript can also show the info on the accepted values already at the editor level.

We can create a `type` using the TypeScript native keyword `type`. Let's describe our type `Operation`:
```
type Operation = 'multiply' | 'add' | 'divide';
```

Now the `Operation` type accepts only three kinds of values; exactly the three strings we wanted. Using the OR operator `|`, we can define a variable to accept multiple values by creating a union type. In this case, we used exact strings (called string literal types), but with unions, you could also make the compiler accept, for example, both string and number: `string | number`.

The `type` keyword defines a new name for a type: a type alias. Since the defined type is a union of three possible values, it is handy to give it an alias that has a representative name.

This is our current calculator:
```
type Operation = 'multiply' | 'add' | 'divide';

const calculator = (a: number, b: number, op: Operation) => {
  if (op === 'multiply') {
    return a * b;
  } else if (op === 'add') {
    return a + b;
  } else if (op === 'divide') {
    if (b === 0) return 'can\'t divide by 0!';
    return a / b;
  }
}
```

Now, when we hover on top of the `Operation` type in the calculator function, we can immediately see suggestions on what to do with it. And if we try to use a value that is not within the `Operation` type, we get the familiar red warning signal and extra info from our editor.

One thing we haven't touched yet is typing the return value of a function. Usually, you want to know what a function returns, and it would be nice to have a guarantee that it returns what it says it does. Let's add a return value `number` to the calculator function:
```
type Operation = 'multiply' | 'add' | 'divide';

const calculator = (a: number, b: number, op: Operation): number => {
  if (op === 'multiply') {
    return a * b;
  } else if (op === 'add') {
    return a + b;
  } else if (op === 'divide') {
    if (b === 0) return 'this cannot be done';
    return a / b;
  }
}
```

The compiler complains straight away because, in one case, the function returns a string. There are a couple of ways to fix this. We could extend the return type to allow string values, like so:
```
const calculator = (a: number, b: number, op: Operation): number | string =>  { 
  // ...
}
```

Or we could create a return type, which includes both possible types, much like our Operation type:
```
type Result = string | number;

const calculator = (a: number, b: number, op: Operation): Result =>  {
  // ...
}
```

But now, the question is if it's really okay for the function to return a string.

When your code can end up in a situation where something is divided by 0, something has probably gone wrong, and an error should be thrown and handled where the function was called. When you are deciding to return values you weren't originally expecting, the warnings from TypeScript can prevent you from making rushed decisions and help you to keep your code working as expected.

One more thing to consider is even though we have defined types for our parameters, the generated JavaScript used at runtime does not contain the type checks. So if, for example, the `Operation` parameter's value comes from an external interface, there is no definite guarantee that it will be one of the allowed values. Therefore, it's still better to include error handling and be prepared for the unexpected to happen. In this case, when there are multiple possible accepted values, and all unexpected ones should result in an error, the switch...case statement suits better than if...else in our code.

The code of our calculator should look something like this:
```
type Operation = 'multiply' | 'add' | 'divide';

const calculator = (a: number, b: number, op: Operation) : number => {
  switch(op) {
    case 'multiply':
      return a * b;
    case 'divide':
      if (b === 0) throw new Error('Can\'t divide by 0!');
      return a / b;
    case 'add':
      return a + b;
    default:
      throw new Error('Operation is not multiply, add or divide!');
  }
}

try {
  console.log(calculator(1, 5 , 'divide'));
} catch (error: unknown) {
  let errorMessage = 'Something went wrong: '
  if (error instanceof Error) {
    errorMessage += error.message;
  }
  console.log(errorMessage);
}
```

## Type narrowing

The default type of the catch block parameter `error` is `unknown`. The unknown is a kind of top type that was introduced in TypeScript version 3 to be the type-safe counterpart of `any`. Anything is assignable to `unknown`, but `unknown` isn't assignable to anything but itself and `any` without a type assertion or a control flow-based narrowing. Likewise, no operations are permitted on an `unknown` without first asserting or narrowing it to a more specific type.

Both the possible causes of exception (wrong operator or division by zero) will throw an Error object with an error message, that our program prints to the user.

If our code would be JavaScript, we could print the error message by just referring to the field message of the object `error` as follows:
```
try {
  console.log(calculator(1, 5 , 'divide'));
} catch (error) {
  console.log('Something went wrong: ' + error.message);
}
```

Since the default type of the `error` object in TypeScript is `unknown`, we have to narrow the type to access the field:
```
try {
  console.log(calculator(1, 5 , 'divide'));
} catch (error: unknown) {
  let errorMessage = 'Something went wrong: '
  // here we can not use error.message
  if (error instanceof Error) {
    // the type is narrowed and we can refer to error.message
    errorMessage += error.message;
  }
  // here we can not use error.message

  console.log(errorMessage);
}
```

Here, the narrowing was done with a typeof type guard. That is just one of the many ways to narrow a type.

## Accessing command line arguments

The programs we have written are alright, but it would be better if we could use command-line arguments instead of always having to change the code to calculate different things.

We can access the command line arguments by accessing `process.argv`, just like in a regular node application. If you are using a recent npm version (7.0 or later), there are no problems, but with an older setup, you will receive an error: *Cannot find name 'process'. Do you need to install type definitions for node? Try `npm i @types/node`.*

## @types/{npm_package}

Let's return to the basic idea of TypeScript. TypeScript expects all globally-used code to be typed, as it does for your code when your project has a reasonable configuration. The TypeScript library itself contains only typings for the code of the TypeScript package. It is possible to write your own typings for a library, but that is rarely needed, since the TypeScript community has done it for us.

As with npm, the TypeScript world also celebrates open-source code. The community is active and continuously reacting to updates and changes in commonly-used npm packages. You can almost always find the typings for npm packages, so you don't have to create types for all of your thousands of dependencies alone.

Usually, types for existing packages can be found from the *@types* organization within npm, and you can add the relevant types to your project by installing an npm package with the name of your package with a `@types/` prefix. For example:
```
npm install --save-dev @types/react @types/express @types/lodash @types/jest @types/mongoose
```

The *@types/\** are maintained by Definitely Typed, a community projet to maintain types of everything in one place.

Sometimes, an npm package can also include its types within the code and, in that case, installing the corresponding *@types/\** is not necessary.

Since the typings are only used before compilation, the typings are not needed in the production build, and they should *always* be in the devDependencies of the package.json.

Since the global variable *process* is defined by Node itself, we get its typings from the package *@types/node*.

Since version 10.0, *ts-node* has defined *@types/node* as a peer dependency. If the version of npm is at least 7.0, the peer dependencies of a project are automatically installed by npm. If you have an older npm, the peer dependency must be installed explicitly:
```
npm install --save-dev @types/node
```

When the package is installed, the compiler does not complain about the variable *process*. Note that there is no need to require the types to the code, the installation of the package is enough.

## Improving the project

Next, let's add npm scripts to run our two programs *multiplier* and *calculator*:
```
{
  "name": "fs-open",
  "version": "1.0.0",
  "description": "",
  "main": "index.ts",
  "scripts": {
    "ts-node": "ts-node",
    "multiply": "ts-node multiplier.ts",
    "calculate": "ts-node calculator.ts"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "ts-node": "^10.5.0",
    "typescript": "^4.5.5"
  }
}
```

We can get the multiplier to work with command line parameters with the following changes:
```
const multiplicator = (a: number, b: number, printText: string) => {
  console.log(printText,  a * b);
}

const a: number = Number(process.argv[2])
const b: number = Number(process.argv[3])
multiplicator(a, b, `Multiplied ${a} and ${b}, the result is:`);
```

And we can run it with:
```
npm run multiply 5 2
```

If the program is run with parameters that are not of the right type, e.g.:
```
npm run multiply 5 lol
```

It "works", but gives us the answer:
```
Multiplied 5 and NaN, the result is: NaN
```

The reason for this is that `Number('lol')` returns `NaN`, which is actually of type `number`, so TypeScript has no power to rescue us from this kind of situation.

To prevent this kind of behavior, we have to validate the data given to us from the command line.

The improved version of the multiplicator looks like this:
```
interface MultiplyValues {
  value1: number;
  value2: number;
}

const parseArguments = (args: string[]): MultiplyValues => {
  if (args.length < 4) throw new Error('Not enough arguments');
  if (args.length > 4) throw new Error('Too many arguments');

  if (!isNaN(Number(args[2])) && !isNaN(Number(args[3]))) {
    return {
      value1: Number(args[2]),
      value2: Number(args[3])
    }
  } else {
    throw new Error('Provided values were not numbers!');
  }
}

const multiplicator = (a: number, b: number, printText: string) => {
  console.log(printText,  a * b);
}

try {
  const { value1, value2 } = parseArguments(process.argv);
  multiplicator(value1, value2, `Multiplied ${value1} and ${value2}, the result is:`);
} catch (error: unknown) {
  let errorMessage = 'Something bad happened.'
  if (error instanceof Error) {
    errorMessage += ' Error: ' + error.message;
  }
  console.log(errorMessage);
}
```

When we now run the program with bad input, we get a proper error message:
```
Something bad happened. Error: Provided values were not numbers!
```

There is quite a lot going on in the code. The most important addition is the function `parseArguments`, that ensures that the parameters given to `multiplicator` are of the right type. If not, an exception is thrown with a descriptive error message.

The definition of the function has a couple of interesting things. First, the parameter `args` is an array of strings. The return value of the function has the type `MultiplyValues`, which is defined as an object containing two values. The definition utilizes TypeScript's *Interface* keyword, which is one way to define the "shape" an object should have. In our case, it is quite obvious that the return value should be an object with the two properties `value1` and `value2`, which should both be of type number.

## The alternative array syntax

Note that there is also an alternative syntax for arrays in TypeScript. Instead of writing
```
let values: number[];
```

we could use the "generics syntax" and write:
```
let values: Array<number>;
```

In this course, we shall mostly be following the convention enforced by the Eslint rule array-simple, that suggests to write the simple arrays with the [] syntax, and use the <> syntax for the more complex ones.

## More about tsconfig

We have so far used only one tsconfig rule, noImplicitAny. Now, it's time to look into the config file a little deeper.

As mentioned, the tsconfig.json file contains all your core configurations on how you want TypeScript to work in your project.

Let's specify the following configurations in our *tsconfig.json* file:
```
{
  "compilerOptions": {
    "target": "ES2022",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitAny": true,
    "esModuleInterop": true,
    "moduleResolution": "node"
  }
}
```

Do not worry too much about the `compilerOptions`; they will be under closer inspection later on.

You can find explanations for each of the configurations from the TypeScript documentation, or from the tsconfig page, or from the tsconfig schema definition.
tsconfig page: https://www.typescriptlang.org/tsconfig
tsconfig schema: http://json.schemastore.org/tsconfig

## Adding Express to the mix

It is time to start working with some HTTP requests. Start by installing Express:
```
npm install express
```

Then, add the `start` script to package.json:
```
{
  // ..
  "scripts": {
    "ts-node": "ts-node",
    "multiply": "ts-node multiplier.ts",
    "calculate": "ts-node calculator.ts",
    "start": "ts-node index.ts"
  },
  // ..
}
```

Now, create the file *index.ts*, and write the HTTP GET * ping endpoint to it:
```
const express = require('express');
const app = express();

app.get('/ping', (req, res) => {
  res.send('pong');
});

const PORT = 3003;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

The `req` and `res` parameters of `app.get` need typing. VSCode is also complaining about the importing of Express. The complaint is that the `'require' call may be converted to an import.` We can follow the advice and write the import as follows:
```
import express from 'express';
```

We will have another problem with the import - we haven't installed types for *express*. We can do so by running:
```
npm install --save-dev @types/express
```

If we use require, anything express-related will be of type `any`. Whereas when we use `import`, the editor knows the actual types. Which import statement to use depends on the export method used in the imported package.

A good rule of thumb is to try importing a module using the `import` statement first. We will always use this method in the *frontend*. If `import` does not work, try a combined method: `import ... = require('...')`.

One more problem in our code is that the editor complains that `'req' is declared but its value is never read`. This is because we banned unused parameters in our *tsconfig.json*.

This configuration might create problems if you have library-wide predefined functions which require declaring a variable even if it's not used at all, as is the case here. Fortunately, this issue has already been solved on the configuration level. If it is absolutely impossible to get rid of an unused variable, you can prefix it with an underscore to inform the compiler you have thought about it and there is nothing you can do.

After renaming `req` to `_req`, everything should work fine with no errors.

To simplify the development, we should enable *auto-reloading* to improve our workflow. We have already used *nodemon*, but ts-node has an alternative called *ts-node-dev*. It is meant to be used only with a development environment that takes care of recompilation on every change, so restarting the application won't be necessary.

Let's install `ts-node-dev` to our development dependencies.
```
npm install --save-dev ts-node-dev
```

And add a script to *package.json*:
```
{
  // ...
  "scripts": {
    // ...
    "dev": "ts-node-dev index.ts",
  },
  // ...
}
```

Now, by running `npm run dev`, we have a working, auto-reloading development environment for our project.

## The horrors of any

Now that we have our first endpoints completed, you might notice we have used barely any TypeScript in these small examples. When examining the code a bit closer, we can see a few dangers lurking there.

Let's add the HTTP POST endpoint `calculate` to our app:
```
import { calculator } from './calculator';

app.use(express.json());

// ...

app.post('/calculate', (req, res) => {
  const { value1, value2, op } = req.body;

  const result = calculator(value1, value2, op);
  res.send({ result });
});
```

