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

The default type of the catch block parameter `error` is `unknown`. The unknown is a kind of top type that was introduced in TypeScript version 3 to be the type-safe counterpart of `any`.