# Deploying app to Internet

## Same origin policy and CORS

Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (eg. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served. Certain "cross-domain" requests, notably Ajax requests, are forbidden by default by the same-origin security policy.

If you try to link a frontend running on localhost port 3000 to a backend running on localhost port 3001, it will cause an error, since they do not have the same origin.

We can allow requests from other *origins* by using Node's cors middleware.

First, install *cors* with `npm install cors`. Import it into your backend code and use `app.use(cors())`, and this will resolve the error.

## Application to the Internet

Websites that could be used to host NodeJS:
Fly.io: https://fly.io/
Render: https://render.com/
Railway: https://railway.app/
Cyclic: https://www.cyclic.sh/
Replit: https://replit.com/
CodeSandbox: https://codesandbox.io/

For Fly.io, we need to change the definition of the port in previous examples:
```
const PORT = process.env.PORT || 3001
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

We need to use the port in the environment variable `PORT` if it is available.

## Fly.io

First, don't forget to change the port you use to `process.env.PORT`, and install and use the `cors` package as well.

Authenticate via the command line with the command: `fly auth login`

Initializing an app happens by running the following command in the root directory of the app: `fly launch`

Give the app a name, or let Fly.io auto-generate one. Pick a region where the app will be run. Do not create a Postgres database for the app, since it is not needed.

The last question is "Would you like to deploy now?" Answer yes, and your app will be deployed to the Fly.io servers.

If it works, you can open your app in the browser with the command: `fly open`

After the initial setup, when the app code has been updated, it can be deployed to production with the command: `fly deploy`

A particularly important command is `fly logs`. This command can be used to view server logs.

Fly.io creates a file *fly.toml* in the root of your app. The file contains all the configuration of your server.

**Note:** In some cases, running Fly.io commands on Windows WSL can cause problems. If the following command hangs: `flyctl ping -o personal`, your computer cannot connect to Fly.io. The following link could contain a possible fix: https://github.com/fullstack-hy2020/misc/blob/master/fly_io_problem.md

If the output looks like below, there are no connection problems:
```
$ flyctl ping -o personal
35 bytes from fdaa:0:8a3d::3 (gateway), seq=0 time=65.1ms
35 bytes from fdaa:0:8a3d::3 (gateway), seq=1 time=28.5ms
35 bytes from fdaa:0:8a3d::3 (gateway), seq=2 time=29.3ms
...
```

## Frontend production build

So far, we have been running React code in *development mode*. In development mode, the application is configured to give clear error messages, immediately render code changes to the browser, etc.

When the application is deployed, we must create a production build, or a version of the application which is optimized for production.

A production build of applications created with *create-react-app* can be created with the command `npm run build`. This creates a directory called *build*, containing the only HTML file of our application, *index.html*, and the directory *static*. A minified version of our JavaScript code will be generated in the static directory. All of the JavaScript will be minified into one file. All code from all the application's dependencies will also be minified into this file.

## Serving static files from the backend

One option for deploying the frontend is to copy the production build to the root of the backend repository and configure the backend to show the frontend's *main page* (the file *build/index.html*) as its main page.

To copy the production build of the frontend to the root of the backend with the command:
```
cp -r <build-folder> <backend-root>
```

To make express show *static content*, the page *index.html* and the JavaScript, etc., it fetches, we need a built-in middleware from express called static. To add it, use the following line of code:
```
app.use(express.static(<build-folder>))
```

When express gets an HTTP GET request, it will first check if the *build* directory contains a file corresponding to the request's address. If a correct file is found, express will return it.

Now HTTP GET requests to the address *www.serversaddress.com/index.html* or *www.serversaddress.com* will show the React frontend. GET requests to the API addresses will be handled by the backend's code.

Because both the frontend and the backend are at the same address in this case, we can declare `baseUrl` as a relative URL in the frontend code. Eg. `'/api/notes'` instead of `'www.serversaddress.com/api/notes'`

## Proxy

After changing the URL in the frontend, it will no longer work in development mode. This is because the requests to the backend will go to *localhost:3000*. To fix this, if the project was created with create-react-app, you only need to add the following declaration to *package.json*:
```
{
  "dependencies": {
    // ...
  },
  "scripts": {
    // ...
  },
  "proxy": "http://localhost:3001"]
}
```

Now, the React development environment will work as a proxy, and when the React code does an HTTP request to a server address at *http://localhost:3000* not managed by the React application itself (ie. not fetching the CSS or JavaScript of the application), the request will be redirected to the server at *http://localhost:3001*.

## Streamlining deploying of the frontend

For Fly.io, add the following scripts to the backend *package.json*:
```
{
  "scripts": {
    // ...
    "build:ui": "rm -rf build && cd <frontend-directory> && npm run build && cp -r build <backend-directory>",
    "deploy": "fly deploy",
    "deploy:full": "npm run build:ui && npm run deploy",    
    "logs:prod": "fly logs"
  }
}
```

The current approach makes it difficult to deploy the frontend. This will make creating an automated deployment pipeline more difficult, and we will look at other methods to fix this later.
