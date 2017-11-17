# Quick Start for Docker Users

If you are already a Docker user, Sandbox will make your life with Docker even easier. This quick start guide is designed to show Docker users how they can use Sandbox to take Docker further for daily development tasks like building applications, and running unit and functional tests.

## Building from source

If you can find an existing Docker image that contains the exact environment you need, you can immediately issue a `docker run` command. However, if you need to make small tweaks to an existing image, you will find yourself having to create a Dockerfile. This is not an uncommon scenario. Consider a quick example Dockerfile that would be used for building Node.js applications: 

```docker
FROM node:8-alpine
RUN apk update && apk add git
```

You can see that while the official Node.js image - `node:8-alpine` - almost servces our purpose, we had to add git to it because one of the package dependencies in the Node.js application needs it.

Because you now have a custom Dockerfile, you will have to first build a Docker image with your Dockerfile before you can run a container with it. When other members of your team need to use the same Dockerfile, they will also have to build the image first before they can run a container with it. Of course, you have a few options to make this simpler. You may push the image to a registry but if you use proprietary code or tools in the image, you will first want to setup a private registry to push your images to. Alternatively, you can write some scripts that take your Dockerfile and perform a `docker build` before performing a `docker run`.

Sandbox makes this easier: when you run a Sandbox workflow, Sandbox takes care of building any images necessary. Sandbox allows you to use your existing Dockerfiles in a workflow so using the Dockerfile above would be rather straightforward:

```yaml
steps:
 - run:
     name: Node.js environment
     dockerfile: Dockerfile
     cache: true
 - run:
     step: Node.js environment
     script: ${args}
```

With that workflow saved as a YAML file called `node.yml`, we can issue a simple command to run the workflow `sbox run node npm install`. The Sandbox CLI will build the image from the Dockerfile as the first step, and then pass all arguments after the workflow name (in this case, that's `npm install`) to the second step. You can see we use a variable placeholder called `${args}` in order to execute whatever arguments were passed as the script for the second step.

An abridged version of the console output when running this workflow looks like this:
```
ravi@dev:~/demo1$ ./sbox run node npm install
Building image for step Node.js environment:
Step 1 : FROM node:8-alpine
...
Build image for step 1:
...
Running step 1:
[Step 1] npm install
...
```

You can see Sandbox shows you the output from building the image from the Dockerfile, and the script that is run in the second step. Note that we can directly do a `npm install` because by default Sandbox adds all files from the project directory into the image at the location `/app` (all of this can be customized).

## Mounting volumes

As a Docker user, you also know that mounting host directories as volumes can be very useful to get artifacts produced in a container. If you use Docker for development tasks, it's a great way to get at build artifacts produced by a build, or test reports from a test run. Normally, when you want to bind mount a host path as a volume in Docker, you add an argument to the `docker run` command. If you do this regularly, it can be frustrating to have to pass in the volume flag.

You might also be frustrated to realize that you cannot use volumes during a `docker build`. There are cases where this is useful - for example, if you are a Java developer, you want to be able to use an existing Maven local repository when building your application, and putting it in a Docker image. You soon realize that you cannot mount the Maven repo as a volume.

Sandbox typically works within the context of a project. That means when you run a workflow, that workflow is generally run within a project directory. This is why Sandbox allows you to specify project-relative paths as volumes to mount in a workflow. Let's add to the example workflow from above:

```yaml
steps:
 - run:
     name: Node.js environment
     dockerfile: Dockerfile
     cache: true
 - run:
     step: Node.js environment
     volumes:
       hostPath: ./dist
       containerPath: /app/dist
     script: ${args}
```

Now, when you run the workflow using a command like `sbox run node npm run build`, it will perform an `npm run build`. In this example, that would build artifacts within the `/app/dist` directory. Because we mounted the `dist` directory inside the project as a volume at that location, we can easily get at the build artifacts produced in the `dist` folder.

## More powerful health checks

Finally, let's talk about health checks. If you use Docker Compose, you know that you can compose together a set of services runinng within their own containers. If you have experience with services, you probably also have experience trying to deal with those services inside containers being ready. For example, if you start a MySQL service within a container, you have to wait for the MySQL service to be ready before you can use it. For those of you with more experience, you probably are aware that you can create health checks for your services using a `HEALTHCHECK` Dockerfile instruction or the `healthcheck` key in a compose file. However, 

If you are a Kubernetes user, you know that Kubernetes offers more advanced health checks for services. Because Sandbox runs a single-node Kubernetes cluster in order to run workflows, the same health checks are available to you. That means, you can very easily add TCP and HTTP-based health checks to your workflows.

Take a look at a simple example of waiting for a MySQL database to start up:

```yaml
steps:
...
 - service:
     image: mysql:5.7
     readiness:
       tcp:
         port: 3306
...
```

The step shown here starts a long-running service using an official Docker MySQL image. It then waits till a connection can be established to TCP port 3306. The workflow will proceed to subsequent steps only after the service is readiness using that health check.

## Reproducible and shareable tasks

Each of the conveniences described above are relatively minor. But taken together, they allow you to think differently about your daily development tasks.

As a Docker user, you know that Docker containers are great for reproducing runtime environments. If you have a Docker image, running a container is as simple as issuing a `docker run` command.

Sandbox takes this further by giving you the following additional conveniences:


 But for reproducing development environments, there are 

 Docker images allow you to capture all of the dependencies, configuration and artifacts your applicaton needs.


Let's start with what you would do to get started with Docker. You would create your own Dockerfile to include the packages.


