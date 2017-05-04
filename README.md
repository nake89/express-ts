# Using Typescript with Node JS

In this article we will set up an [Express][1] application using [Typescript][2].

## Requirements

In order to follow this tutorial you will need the latest version of [Node js][3], Typescript, and [Typings][4] installed.
I recommend installing node js via [Homebrew][5] if you are on OSX.
Once node is installed, both Typings and Typescript can be installed via [NPM][6].

Use the following commands to do so:

```
npm install -g typescript
npm install -g typings
```

Optionally you should install [Visual Studio Code][7].
This is a code editor from Microsoft that is built using Typescript.
It provides an excellent environment for working with Typescript and many other programming languages.

## Setting Up a Typescript Project

Now that the prerequisites are installed we can begin setting up the Typescript project.
Open up a terminal, create, and cd into a directory called _express-ts_.

```
mkdir express-ts
cd express-ts
```

Now that we are in the created directory we can initialise the Typescript project.
To do this we will use the `tsc` and `typings` commandline executables.
These were installed via Typescript and Typings respectively.

Firstly we want to create a `tsconfig.json` file. This can be done with `tsc`.

```
tsc --init
```

Running the above command creates the `tsconfig.json` in the current directory.
It also adds some useful boilerplate code to the file.
Looking inside the newly generated `tsconfig.json` you should see the following.

```
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es5",
        "noImplicitAny": false,
        "sourceMap": false
    },
    "exclude": [
        "node_modules"
    ]
}
```

For the purposes of this demo we are going to put the compiled `.js` files into a `build` directory.
In order to tell the Typescript compiler that this is where the compiled files should go we must add an `outDir` parameter.
Add the following to the `compilerOptions` object inside the `tsconfig.json` file.

```
"outDir": "build"
```

Now that the `tsconfig` is correctly set up we will add some Typescript Definition files or `.d.ts` files via Typings.
These files are used to give the compiler knowledge of the application.

Run the following to create a `typings.json` file.

```
typings init
```

As we will be using Express and ES2015 syntax. We need to install the `es6-shim`, `node`, and `express` typings.
Run the following commands to do so.

```
typings install dt~node --save --global
typings install dt~es6-shim --save --global
typings install dt~express --save --global
typings install dt~serve-static --save --global
typings install dt~express-serve-static-core --save --global
typings install dt~mime --save --global
```

The above commands will update the `typings.json` file which was generated by the `typings init` command and place the Typings dependencies in the typings folder.

Now we can install the application depencies via NPM.
In the application root folder run the following.

```
npm init -y
npm install --save express
```

This creates a `package.json` file and installs Express as a dependency.

Finally lets create the application files.
Make a folder called `app` and add a `server.ts` file.
The application will be a _greeter_. So lets also add a `WelcomeController`.
Inside the app folder create another folder called `controllers` and a `welcomeController.ts` and `index.ts` file.

```
mkdir app && cd app
touch server.ts
mkdir controllers && cd controllers
touch index.ts welcomeController.ts
```

The application should now have the following structure.

```
.
├── app
│   ├── controllers
│   │   ├── index.ts
│   │   └── welcomeController.ts
│   └── server.ts
├── node_modules
├── package.json
├── tsconfig.json
├── typings
└── typings.json

```

## Creating An Express App

As mentioned above we will be creating a greeting app.
This app will have one route that takes a `name` parameter and then greets that name.

Open a text editor inside the application folder.
If you are using Visual Studio Code you can open it inside the folder by running the `code` command with a folder argument.

```
code .
```

Firstly lets take a look at the `welcomeController.ts` file.
This file will handle the welcome routes.
To do this we need it to export an Express router object.
I have added code comments to the snippet below which explains how this file should work.

```javascript
/* app/controllers/welcomeController.ts */

// Import only what we need from express
import { Router, Request, Response } from 'express';

// Assign router to the express.Router() instance
const router: Router = Router();

// The / here corresponds to the route that the WelcomeController
// is mounted on in the server.ts file.
// In this case it's /welcome
router.get('/', (req: Request, res: Response) => {
    // Reply with a hello world when no name param is provided
    res.send('Hello, World!');
});

router.get('/:name', (req: Request, res: Response) => {
    // Extract the name from the request parameters
    let { name } = req.params;

    // Greet the given name
    res.send(`Hello, ${name}`);
});

// Export the express.Router() instance to be used by server.ts
export const WelcomeController: Router = router;
```

Now that the `WelcomeController` is ready to be used lets export it from the _controllers_ folder.
As you may know `index` files act as folder entry points.
Thanks to this we can access exports from the `index.ts` file we created in the _controllers_ folder.
Add the following to that file.

```javascript
/* app/controllers/index.ts */
export * from './welcomeController';
```


**Note:** as good practice you should never add application logic inside an index file.

Now that the `WelcomeController` is being correctly exported lets make use of it inside the `server.ts` file.


```javascript
/* app/server.ts */

// Import everything from express and assign it to the express variable
import * as express from 'express';

// Import WelcomeController from controllers entry point
import {WelcomeController} from './controllers';

// Create a new express application instance
const app: express.Application = express();
// The port the express app will listen on
const port: number = process.env.PORT || 3000;

// Mount the WelcomeController at the /welcome route
app.use('/welcome', WelcomeController);

// Serve the application at the given port
app.listen(port, () => {
    // Success callback
    console.log(`Listening at http://localhost:${port}/`);
});
```

Now that we have finished writing the application lets transpile it to javascript.
In the root of the application run the following.

```
tsc
```

Alternatively you may run the following to tell the Typescript compiler to run everytime it detects a filesystem change.

```
tsc --watch
```

These commands will tell the Typescript compiler to build the application based on the `tsconfig.json`.
If everything has been successfull you will see a newly created `build` directory.
This is where the Typescript compiler has placed the generated `.js` files based of the optional `outDir` parameter in the `tsconfig`.

We can now run the app with the following command.

```
node build/server.js
```

Open a browser at the following url [http://localhost:3000/welcome](http://localhost:3000/welcome).
You should be greeted with a _Hello, World!_.
Try visiting [http://localhost:3000/welcome/Borris](http://localhost:3000/welcome/Borris) to confirm the
`WelcomeController` is functioning correctly.
You should see a _Hello, Borris!_.


## Summary

This application, though simple, gives a good basis for creating future Express applications with Typescript.

You have learned how to intialise a Typescript application from the command line.
You have also seen how to make use of Typings within a Typescript application.
Learning how to use typings is very important when working with Typescript as they give the compiler a better knowledge of the application.
The more the compiler knows, the better it can help you out.

You have also learned a bit about structuring an Express application.
Many Express tutorials bundle code into one big `server.js` file.
This leads to an application that is very difficult to maintain and is not representative of real world applications.
However we have seen here that it is possible to _mount_ Express Router instances so that applications can be broken down into modular parts.
This leads to more maintainable and scalable applications.

## Next Steps?

Working with data is an important skill to learn when using Node.
So, with the knowledge gained here I would recommend investigating using Express and Typescript with the [Sequelize][8] ORM.
It is an awesome NPM package which lets a Node app easily communicate with an SQL database.
This will allow you to create dynamic and data driven Node apps.


---

_Finished application files can be found on github at the below link._


[https://github.com/BrianDGLS/express-ts](https://github.com/BrianDGLS/express-ts)


[1]: http://expressjs.com/
[2]: https://www.typescriptlang.org/
[3]: https://nodejs.org/en/
[4]: https://github.com/typings/typings
[5]: http://brew.sh/
[6]: https://www.npmjs.com/
[7]: https://code.visualstudio.com/
[8]: http://docs.sequelizejs.com/en/v3/
