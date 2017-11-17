## Quick Start

{className:alert}This quick start guide will quickly introduce you to Sandbox, and get you started running workflows in a matter of minutes.

You can also watch a short [Introduction to Sandbox](https://www.youtube.com/watch?v=RwMl-vy-1Vs) video, which covers many of the same concepts as this quick start guide.

## Running your first workflow []('#run')

One of the key benefits Sandbox gives you is the ability to run automated workflows straight from Git, without needing to install any other software. You can experience this for yourself by cloning a Git repository we created to serve as an example.

To get started, clone our example repository:≈

```bash
git clone https://github.com/stackfoundation/sbox-mean
```

You now have a Sandbox-enabled project! Without installing any software, you can immediately run a containerized version of the example application in the repository you just cloned by running the `dev-server` workflow:

```bash
cd sbox-mean
./sbox run dev-server
```

This will do the following:

*   Bootstrap Sandbox, downloading and installing some additional CLI components automatically. [See what components Sandbox uses in the Architecture section](/docs/overview#architecture)
*   Execute the `run` command, which:
    *   Sets up and starts the Sandbox virtual machine (this happens only on the first run). This may install some additional software to allow running a virtual machine on your operating system - Sandbox will handle this automatically.
    *   Runs a workflow named `dev-server`, which is included in the repository.

The `dev-server` workflow starts a MongoDB database, an HTTP server, and an example web application (within containers). You will be able to watch as the example application starts up because Sandbox will show you console output during the workflow execution. Because the `dev-server` workflow launches a long-running application, the application will stay open until the workflow is manually interrupted (using `CTRL` + `C`).

## {translate('TITLE_START_VM')}

<a id="{'vm'}"></a>

Now that the `dev-server` workflow started an application, we need to be able to access it. Sandbox will start the containers on the Sandbox virtual machine (VM), and so the ports opened by the application will be opened on that VM's IP. So, let's first get the IP of the VM - in a new terminal window, run:

```bash
sbox status
```

This will return the status of the Sandbox VM, and the address of the Kubernetes Dashboard, at `http://{'{'}Sandbox VM IP{'}'}:30000`.  

With this IP, we can now access `http://{'{'}Sandbox VM IP{'}'}:31000`, which is the port the application opened. If you open it in your browser, you should see this:

![](assets/images/docs/getting-started-app-result.png)

That's it! You have now connected to the containerized web application that Sandbox started! Notice that you had to perform no setup or configuration - Sandbox performed all the work necessary to setup the environment for the application. It was all part of Sandbox running your first workflow.

## {translate('TITLE_START_WORKFLOW')}

<a id="{'workflow'}"></a>

The `dev-server` workflow that we just ran can be found at `{'{'}project root{'}'}/workflows/dev-server.wflow`. It's just a YAML file:

```yaml
steps:
  - name: Launch Mongo
    [...]
  - name: Install application
    [...]
  - name: Run application
    [...]
```

Workflows are just sequential list of steps to run. This particular workflow has three steps: `Launch Mongo`, `Install application` and `Run application`. We'll take a look at these steps in a little more detail in the sections that follow.

Workflow files are explained in more detail [in their own dedicated section](/docs/workflows).

## {translate('TITLE_START_SERVICES')}

<a id="{'services'}"></a>

The first step - the `Launch Mongo` step - is a [service step](/docs/workflows#services). Let's take a look at it in more detail:

```yaml
- name: Launch Mongo
  image: 'mongo:jessie'
  ports:
    - name: mongo
      containerPort: 27017
      internalPort: 27017
  type: service
  readiness:
    type: tcp
    port: 27017
```

### **image** field

Defines the Docker image to use - here, we are using a Docker image with MongoDB pre-installed.

### **type** field

Specifies that this step is a service step, meaning it starts a long-running service that can be accessed by other steps. In practice, this means that Sandbox will not wait for this step to complete before moving on - service steps are generally meant to run indefinitely, as is the case for this database service.

### **readiness** field

Used to define when the service should be considered ready. In this case, we assume the service to be ready as soon as we can connect to port `27017`.

### **ports** field

Defines ports to be exposed. For this service, we expose port `27017` internally to the cluster under the same port number. We also specify the name `mongo` to the port, to allow it to be discovered by other steps, as we'll see in the next step definition.

## {translate('TITLE_START_CACHING')}

<a id="{'caching'}"></a>

The second step - the `Install application` step - is [cached](/docs/workflows#cache), and will not be re-run if no changes occur to the relevant project source files. Let's examine this step:

```yaml
- name: Install application
  image: 'node:8-alpine'
  cache: true
  dockerignore: dockerignoreForNpmInstall
  script: |-
    apk update && apk add git
    cd app
    npm install
```

### **image** field

As before, defines a Docker image to use - here, we are using an image with Node.js pre-installed.

### **type** field

Since it is not present, it defaults to `sequential`, meaning this step will run until completion, blocking any further steps.

### **cache** field

Enables caching for this step - if no changes occur to the relevant project source files, the step will not be run again.

### **script** field

Defines what gets run in the step. In this case, we are running `npm install`, and installing some other dependencies specific to this project using `apk`. This is a lengthy process, which is why this step is cached.

### **dockerignore** field

Defines a specific file to use as the `dockerignore` file for this step. The `dockerignore` file defines which project source files should be excluded when building the image for this step (normally, they are all included). This is important in this step, as the files included will determine cache validity. The `dockerignoreForNpmInstall` file referenced here excludes most of the project source files, except for `package.json` so that only changes to `package.json` will invalidate the install step cache.

## {translate('TITLE_START_PREVIOUS')}

<a id="{'previous'}"></a>

The final step - the `Run application` step - uses the final state of the container from the [previous step as it's starting image](/docs/workflows#previous). Let's look at this step:

```yaml
- name: Run application
    image: Install application
    imageSource: step
    script: |-
      cd app
      export PORT=31000
      export HOST='0.0.0.0'
      export MONGO_HOST='mongodb://mongo/new-mean'
      export MONGO_PORT=27017
      npm run start
    ports:
      - containerPort: 4040
        internalPort: 31001
        externalPort: 31001
      - containerPort: 31000
        internalPort: 31000
        externalPort: 31000
```

### **image** field

References a previous step - the `Install Application` step. We use a (cached) image with all dependencies pre-installed. In order to tell Sandbox this is a reference to a previous step, we need to set the `imageSource` field to `step` (it defaults to `image`).

### **type** field

Again, defaults to `sequential`, meaning it will run until completion and block the workflow from continuing.

### **script** field

Exports some environment variables: the application and MongoDB service IPs and ports. Here, we connect to the MongoDB service using the URI `mongodb://mongo/new-mean`. Notice that we use `mongo` as the host, which we defined in the exposed port for the Mongo service step. The `name` property in exposed ports allows different steps to discover services and communicate.

### **ports** field

Defines the following ports to be exposed:

*   Port `31000` exposed as `31000` - this is our application's front-end HTTP port.
*   Port `4040` exposed as `31001` - this is a port the application uses to run an API server that interacts with the database.

Notice that these ports don't have `name` fields set. This is because they are only intended to be used as external ports, so no DNS resolution is necessary.

Workflows and all the fields describe above are explained in more detail in the [Workflows](/docs/workflows) section.