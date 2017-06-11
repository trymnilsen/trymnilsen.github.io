---
layout: post
title: Setting up Babylon JS development environment with typescript and vscode
excerpt: "Create a complete "
categories: ["Game Development"]
tags: ["babylonjs","typescript", "vs code"]
comments: true
image:
  feature: https://d33wubrfki0l68.cloudfront.net/4cb4a7f2f373f428ff0628d1aba43d4654983c9e/bdf96/img/layout/logo-babylonjs.svg
---

Babylonjs is a cool framework enabling simple use of WebGL to create 3D games in the browser.
Getting started with BabylonJS is really easy as there is already a playground at [http://www.babylonjs-playground.com](http://www.babylonjs-playground.com/).

In this post however we will set up a complete local development enviroment using NodeJS and VSCode

What we will end up with is this scene

![Final scene](https://i.imgur.com/XCMoLNx.png)

### Download everything needed

First head on off to [Node Webpage](https://nodejs.org) and [VSCode](https://code.visualstudio.com/) and download both of them. 

### Package manager setup

We will be using npm for handling packages for us, npm is invoked with the simple `npm` command.
Npm uses a `packages.json` file to keep track of packages, run 

```
    npm init
``` 

to have npm set up a file for you. Follow the instructions in the console.

We will be needing the following packages

* babylonjs  (The 3D library)
* concurrently (Simplifies both running our http server and watching for typescript changes)
* serve (A simple http server, allowing us to see our changes in the browser)
* systemjs (A module loader. Makes using compiled typescript module files a dream)
* typescript (The typescript compiler)

These can either be installed with `npm install <packagename>` or added to the `package.json` file.
If you add the entries directly to the package.json file, remember to run `npm install` afterwards to pull them down.

> A Note on versions: Babylon 2.5 has some issues working with some of the newer defintion files of Typescript. 2.1.5 works fine.

My `package.json` file looks like this after an install

```
    {
      "name": "babylonjs-typescript-demo",
      "version": "1.0.0",
      "description": "babylonjs typescript demo",
      "main": "index.html",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "dependencies": {
        "babylonjs": "^2.5.0",
        "concurrently": "^3.4.0",
        "serve": "^5.2.1",
        "systemjs": "^0.20.10",
        "typescript": "~2.1.5"
      },
      "author": "",
      "license": "MIT"
    }

```

### Lets build the scaffolding

We will need some files to view our webpage.
Paste this into the following files.

**index.html**
{% highlight html %}
    <!DOCTYPE html>
    <head>
      <title>BabylonJS demo</title>
      <link rel="stylesheet" type="text/css" href="css/style.css" />
      <script src="/node_modules/babylonjs/babylon.js"></script>
        <script src="/node_modules/systemjs/dist/system.src.js"></script>
        <script src="/js/systemjs.config.js"></script>
        <script>
          System.import('app').then(function() {
              console.log('Imported');
          }).catch(function(err){ 
              console.error(err); 
            });
        </script>
    </head>
    <body>
        <canvas id="renderCanvas"></canvas>
    </body>
    </html>
{% endhighlight %}

**css/style.css**

{% highlight css %}
    html, body {
        height: 100%;
        width: 100%;
        padding: 0;
        margin: 0;
        overflow: hidden;
    }
    canvas {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
    }
{% endhighlight %}


Your directory structure now looks like this 

```
    yourAwesomeBabylonProject
        |- node_modules/
        |- css/style.css
        |- index.html
        |- package.json
```

### Typescript compilation

Options to the typescript compiler can either be provided in a `tsconfig.json` or to the commandline as flags.

Our typescript file will look like this

```
    {
      "compilerOptions": {
        "target": "ES5",
        "module": "system",
        "moduleResolution": "node",
        "outDir": "js/build",
        "rootDir": "./ts"
        "noEmitOnError": true
      },
      "exclude": [
        "node_modules",
        ".npm"
      ],
      "include": [
            "ts/**/*.ts"
    ]
    }
```

Here we compile down to `ES5` using the `systemjs` module format. We will emit the compiled files into `js/build` if there is no errors. We tell the typescript compiler that our ts folder is the root, this makes the compiler not collapse the directory structure. 

If we have a `.ts` file located at `ts/folder1/folder2/myfile.ts` it will be placed at `js/myfile.js` without any root dir. With root dir defined our compiled filed is placed a `js/folder1/folder2/myfile.js`.

We exclude any files in `node_modules` and `.npm`.
Any files in the `ts` folder ending in `.ts` is included in the compilation.

### SystemJS

We use the [SystemJS](https://github.com/systemjs/systemjs) module loader to fetch and execute our compiled typescript modules. SystemJS requires a config file to know where and what to load. You'll notice it has been included in our `index.html` file as `js\systemjs.config.js`. As we have previously told typescript to compile down to systemjs modules these can be easily included.

It looks like this

**js/systemjs.config.js**

{% highlight javascript %}

    (function (global) {
      System.config({
        map: {
          // our app is within the js/build folder
          app: '/js/build'
        },
        // packages tells the System loader how to load when no filename and/or no extension
        packages: {
          app: {
            main: './game.js',
            defaultExtension: 'js'
          }
        }
      });
    })(this);

{% endhighlight %}

### Creating our Game logic

Create a `game.ts` file in a `ts` folder and paste our skeleton into it

**ts/game.ts**
{% highlight typescript %}

    class Game {
      constructor(canvasElement : string) {
      }

      createScene() : void {
      }

      run() : void {
      }
    }


    // Create our game class using the render canvas element
    let game = new Game('renderCanvas');

    // Create the scene
    game.createScene();

    // start animation
    game.run();

{% endhighlight %}

To get typescript types for all of the babylonjs library we need to add the following line to the top of our game file
```
    /// <reference path="../node_modules/babylonjs/babylon.d.ts" />
```
>The Babylonjs 3.0 Alpha enables us to use the library via imports together with a module loader like systemJS. 
>With 2.5 its easiest to reference the definition file

Its a bit empty now so lets add the instance variables that we will need for our game to run.
Add the following lines to our Game class


{% highlight typescript %}
    class Game {
        private canvas: HTMLCanvasElement;
        private engine: BABYLON.Engine;
        private scene: BABYLON.Scene;
        private camera: BABYLON.FreeCamera;
        private light: BABYLON.Light;
        

    }
{% endhighlight %}

To initialize the Babylon engine we create a instance of it in our game constructor
{% highlight typescript %}
    // Create canvas and engine
    constructor(canvasElement: string) {
      this.canvas = document.getElementById(canvasElement);
      this.engine = new BABYLON.Engine(this.canvas, true);

      // Listen for browser/canvas resize events
      window.addEventListener("resize", ()=> {
          this.engine.resize();
      });
    }
{% endhighlight %}

We're almost there lets set up our scene as well
{% highlight typescript %}
  createScene() : void {
    // We need a scene to create all our geometry and babylonjs items in
    this.scene = new BABYLON.Scene(this.engine);

    // Create a camera, and set its position to slightly behind our meshes
    this.camera = new BABYLON.FreeCamera('freeCamera', new BABYLON.Vector3(0, 5,-10), this.scene);

    // Make our camera look at the middle of the scene, where we have placed our items
    this.camera.setTarget(BABYLON.Vector3.Zero());

    // Attach the camera to the canvas, this allows us to give input to the camera
    this.camera.attachControl(this.canvas, false);

    // Create lightning in our scene
    this.light = new BABYLON.HemisphericLight('skyLight', new BABYLON.Vector3(0,1,0), this.scene);

    // Finally time to add some meshes
    // Create sphere shape and place it above ground
    let sphere = BABYLON.MeshBuilder.CreateSphere('sphere',{segments: 16, diameter: 2}, this.scene);
    sphere.position.y = 1; //not a magic number, but half or our diameter and height

    // Make a plane on the ground
    let ground = BABYLON.MeshBuilder.CreateGround('groundPlane',{width: 6, height: 6, subdivisions: 2}, this.scene);
  }
{% endhighlight %}

Good job! Only thing left to do now is to fill in the `run` method to start the rendering loop.
Add this last part to the `run` method

```
    this.engine.runRenderLoop(()=> {
      this.scene.render();
    });
```

## Running our game

So, how do we run this thing?

Add these tasks to the `scripts` section of your `package.json` file.
```
    "tsc": "tsc -w",
    "http": "serve",
    "start": "concurrently -k -p \"[{name}]\" -n \"HTTP,TSC\" -c \"bgBlue.bold,bgMagenta.bold\" \"npm run http\" \"npm run tsc\""
```

Now you should be able to start your application using `npm run start` and aim your browser at `http:\\localhost:5000` to see the result.

 
