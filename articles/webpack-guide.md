# Hot Reload with webpack and Sandbox [](#getting-started)

In this guide we'll be exploring how to leverage containers using the _Sandbox_ tool when developing a javacript application using _webpack_, and focusing on creating a **usable and useful development platform** using _Sandbox_ - a contanerizzed solution you can use while developing your app, with hot reload.

Most of these examples are applicable to all situations, though. We'll be using as a base webpack's [Getting Started Guide] (https://webpack.js.org/guides/getting-started/), modified to add _webpack-dev-server_, a common tool used when developing using webpack.

## Project Setup [](#setup)

Let's get the webpack stuff out of the way first - we need to setup a project that will use both `webpack` and `webpack-dev-server`. You can follow the instructions in this page, or alternatively get the [Project base as a zip file](). If you download and extract the zip file, you can [skip this part entirely]()

Initialize the project 
```bash
mkdir sbox-webpack-demo && cd sbox-webpack-demo
npm init -y
npm install --save-dev webpack webpack-dev-server copy-webpack-plugin
npm install --save lodash
```

Create the file structure:
```diff
  sbox-webpack-demo
  |- package.json
+ |- index.html
+ |- /src
+   |- index.js
```

index.html
```html
<html>
  <head>
    <title>Getting Started With Sandbox and Webpack</title>
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
```

src/index.js
```javascript
import _ from 'lodash';

function component() {
  var element = document.createElement('div');

  // Lodash, currently included via a script, is required for this line to work
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

webpack.config.js
```javascript
const path = require('path');
var CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  plugins: [
    new CopyWebpackPlugin([
        {
            from: './index.html', to: path.resolve('./dist/index.html'),
        },
    ]),
  ],
  devServer: {
    contentBase: path.join(__dirname, "dist"),
    port: 31000,
    host: '0.0.0.0',
    disableHostCheck: true
  }
};
```

package.json
```diff
  "main": "index.js",
  "scripts": {
+   "build": "webpack",
+   "start": "webpack-dev-server"
-   "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
```

## Adding Sandbox to your project [](#add-sandbox)

Adding Sandbox to your project is as simple as extracting the contents of the cli zip onto your project - [you can get that in our Downloads section](/downloads).

Alternatively, if you have sandbox installed globally in your machine, running `sbox init` in your project's root will add sandbox to it. **Installing `sbox` globally is never required, you only need to have the wrapper files in your project for it to work.**

## Running your first workflow [](#first-workflow)

Our project now builds and runs using `npm start`, with the server running at port `8080`. However, we want to be able to run this inside a container. For that we will need to add a workflow. Sandbox uses workflow files that define a sequence of steps - each step is a Docker container which runs a script.

On a project that uses Sandbox, all workflows exist inside a folder called `workflows` at the root of the project. Let's create our first workflow:

workflows/dev-server.yml
```yaml
steps:
  - run:
      name: Install and run application
      image: 'node:8-alpine'
      script: |-
        cd app
        npm install
        npm start
      ports:
        - name: app-port
          internalPort: 31000
          externalPort: 31000
          containerPort: 31000
          protocol: tcp
```

There is a lot going on here, so let's break it down:

- the workflow contains a sequence of steps (in this case, just one)
- The step defined here is a **run** step, meaning it will run the script defined in the **script** property. A run step will run to completion, so because the process for webpack-dev-server doesn't close, neither will your workflow, unless you exit manually.
- The step needs a base image to use, and we have settled on `node:8-alpine` - this is a docker hub image that is prepared to run node applications.
- Our code is automatically copied to the image by default, under `/app`, so we can `cd` into that directory and run our commands
- We map our ports to external ones, as this is not running in our local machine, but inside an isolated container

### Handling ports on a webpack-dev-server [](#first-workflow-ports)

```js
devServer: {
    contentBase: path.join(__dirname, "dist"),
    port: 31000,
    host: '0.0.0.0',
    disableHostCheck: true
  }
```

You might've noticed some specific setup in `webpack.config.js`, that may seem unusual:
- When exposing ports, the available range is `30000-32767`. This is why our workflow exposes the running app at `31000`. This port **does not need to be the same as the internal one**, but to simplify connecting (and to allow webpack-dev-server to do hot-reload), we set it up to be `31000` both locally to the container, and externally.
- `host` is set to `0.0.0.0`. This is to bind our server port to all interfaces of the container. Remember, the container's `localhost` is different from your machine's `localhost`
- `disableHostCheck` is set to true. This is necessary because the public IP used to connect to our running instance will differ from `0.0.0.0`

### Running your app [](#first-workflow-running)

Now all you need to do to is run your newly created workflow: `./sbox run dev-server`. But we still need to access it.

### Accessing the Sandbox VM [](#first-workflow-vm-ip)

Now that the dev-server workflow started an application, we need to be able to access it. Sandbox will start the containers on the Sandbox virtual machine (VM), and so the ports opened by the application will be opened on that VM's IP. So, let's first get the IP of the VM - in a new terminal window, run:

```bash
sbox status
```

This will return the status of the Sandbox VM, and the address of the Kubernetes Dashboard, at `http://{Sandbox VM IP}:30000`. With this IP, we can now access `http://{Sandbox VM IP}:31000`, which is the port the application opened.

### Using Cache [](#first-workflow-cache)

You might have noticed a problem with the running of this workflow: **it runs `npm install` every time**. Now, for this example, that isn't much of an issue, but as the app grows, so will the number of dependencies, and the time lost running this step. We need to be able to cache the result of the installation step, so that subsequent runs don't need to re-run it unless changes to the dependencies were added. We'll need to change our workflow accordingly:

workflows/dev-server.yml
```diff
steps:
+ - run:
+     name: Install application
+     image: 'node:8-alpine'
+     cache: true
+     source:
+       include:
+         - package.json
+     script: |-
+       cd app
+       npm install
  - run:
-     name: Install and start application
+     name: Run application
-     image: 'node:8-alpine'
+     step: Install application
+     source:
+       include:
+         - '*'
+       exclude:
+         - node_modules
      script: |-
        cd app
-       npm install
        npm start
      ports:
        - name: app-port
          internalPort: 31000
          externalPort: 31000
          containerPort: 31000
          protocol: tcp
```

We are now setting up and using an image prepared with our installed packages as the base to our run step. Let's review the above changes in detail:

* We've added a new step, called **Install Application**:
  * It uses a `source` object, which defines an `include` property. It overrides the default behaviour of including all files, and defines **only the `package.json` file to be included**.
  * It sets `cache` to true - While the image and included files are the same, it uses the cached reult instead of rebuilding each time. Because only `package.json` is included, only changes to this file will trigger the install step to rebuild.
* We've changed our old step:
  * It no longer has an `image` field, but rather a `step` field, set to the previous step, `Install application`. This is the image we had before, `node:8-alpine`, but with the **installed node_modules already included**.
  * It also uses a `source` object, but now including all files, **except `node_modules`** - this is so as to not replace the build `node_modules` with any existing local one

When we run this workflow for the first time, it will install dependencies. However, any subsequent runs will skip this step and use a cached image instead.

### Allowing webpack-dev-server hot reload [](#first-workflow-hot-reload)

One of the advantages of using `webpack-dev-server` is that changes to our code are instantly reflected in the browser. However, if we try that now in our project, nothing will happen. Worse still, webpack in our running workflow **does not recompile at all**. This is understandable - files from our project were **copied** when the workflow was run, so they are detached from your local project once the image is running. We need to fix this, then:

webpack.config.js
```diff
devServer: {
  contentBase: path.join(__dirname, "dist"),
  port: 31000,
  host: '0.0.0.0',
  disableHostCheck: true,
+ watchOptions: {
+     poll: true
+ }
}
```
workflows/dev-server.yaml
```diff
  - run:
      name: Run application
      step: Install application
      source:
        include:
          - '*'
        exclude:
+         - src
          - node_modules
+     volumes:
+       - mountPath: /app/src
+         hostPath: ./src
      script: |-
        cd app
        npm run build
```

What changed here:

- Our workflow step now **excludes** the `src` directory, and instead mounts a volume. Volumes allow us to connect our machine's filesystem to the container's - meaning the files will be shared, and any changes on the host will be reflected on the container.
- `webpack.config.js` needs to have `watchOptions.poll` be true. This is due to the fact that the default watching method does not work in virtual machines.

With these changes, our `dev-server` workflow is fully functional.

## Adding a build workflow [](#build-workflow)

Lastly, we'll need to build our application. This is a simple workflow, very similar to the last one. The only difference is the usage of `volumes` - here we ignore `dist` and instead mount it as a volume, not to update the image's contents, but the host's. The built app will be saved in the `dist` directory, and as that is a volume, will persist those changes to our project's `dist` directory.

```yaml
steps:
  - run:
      name: Install application
      image: 'node:8-alpine'
      cache: true
      source:
        include:
          - package.json
      script: |-
        cd app
        npm install
  - run:
      name: Build application
      step: Install application
      source:
        include:
          - '*'
        exclude:
          - dist
          - node_modules
      volumes:
        - mountPath: /app/dist
          hostPath: ./dist
      script: |-
        cd app
        npm run build
```