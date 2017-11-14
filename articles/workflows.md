# Workflows
Sandbox runs workflows that are comprised of a series of steps, where each step runs a script within a Docker container. Because scripts are run within a Docker container, they have a predictable and well-known environment in which to run, and thus run consistently. Workflows allow you to consistently reproduce all your development tasks.

## Workflow files [](#files)
Workflows are specified as YAML files, with a `.yml` extension, and generally stored within a folder called `workflows`, within a project. Workflows can be manually edited using any text editor (since they are just YAML files). Workflows are run using the [Sandbox CLI](/docs/cli).

Workflow files have the following keys at the top level:

| Key                  | Type                 | Description                              |
| -------------------- | -------------------- | ---------------------------------------- |
| `steps` *(Required)* | Sequence of mappings | Steps that make up the workflow - see [steps](#steps) |
| `ignoreErrors `      | Mapping              | Defines global error handling - see [error handling](#errors) |
| `variables`          | Sequence of mappings | Global variables for the workflow - see [variables](#variables) |

## Steps [](#steps)

Each step within a workflow should have *exactly one* of the following keys defined, and the key defines that step's type:

| Key         | Type    | Description                              |
| ----------- | ------- | ---------------------------------------- |
| `run`       | Mapping | Step that runs from start to finish - see [run steps](#run) |
| `service`   | Mapping | Step that runs a long-running service - see [services](#services) |
| `compound`  | Mapping | Step that waits for sub-steps to be ready or complete - see [compound steps](#compound) |
| `external`  | Mapping | Step that runs an external workflow - see [calling external workflows](#external) |
| `generator` | Mapping | Step that generates a workflow and runs that generated workflow - see [running generated workflows](#generated) |

The following example shows how steps of different types are specified:

```yaml
steps:
 - service:
     name: Launch Mongo
     image: mongo:jessie
     ...
 - run:
     name: Install application
     image: node:8-alpine
     ...
 - service:
     name: Run application
     image: node:8-alpine
```

## Run steps [](#run)

Run steps are the most basic type of step in a workflow. For each run step, Sandbox creates and runs a Docker container, and runs a script inside that container. Run steps are configured using the following keys:

| Key             | Type                 | Description                              |
| --------------- | -------------------- | ---------------------------------------- |
| `name`          | String               | A name for the step                      |
| `ignoreErrors ` | Mapping              | Defines how errors are handled for the step - see [error handling](#errors) |
| `image`         | String               | Docker image to use as base image for the step - see [step image](#image) |
| `step`          | String               | The name of a previous step in the workflow, whose final state to use as the base image for the step - see [step image](#image) |
| `source`        | Mapping              | Configure source file availability for the step - see [source files](#source) |
| `script`        | String               | The actual content of the script to run inside the Docker container created for the step |
| `cache`         | Boolean              | Whether to cache the step image (defaults to `false`) - see [caching step images](#cache) |
| `parallel`      | Boolean              | Whether to run the step in parallel (defaults to `false`) - see [parallel steps](#parallel) |
| `environment`   | Sequence of mappings | Environment variables to make available to the step's container - see [environment variables](#environment) |
| `volumes`       | Sequence of mappings | Volumes to mount on the step container - see [volumes](#volumes) |
| `dockerfile`    | String               | Path to a Dockerfile to use for the step - see [using Dockerfiles](#dockerfiles) |

## Services [](#services)

Service steps are for creating long-running services. Sandbox will not wait for service steps to complete before continuing to the next step. However, it will wait for the service to be ready (see [service readiness](#readiness)) before continuing. Service steps are configured using the following keys:

| Key             | Type                 | Description                              |
| --------------- | -------------------- | ---------------------------------------- |
| `name`          | String               | A name for the step                      |
| `ignoreErrors ` | Mapping              | Defines how errors are handled for the step - see [error handling](#errors) |
| `image`         | String               | Docker image to use as base image for the step - see [step image](#image) |
| `step`          | String               | The name of a previous step in the workflow, whose final state to use as the base image for the step - see [step image](#image) |
| `source`        | Mapping              | Configure source file availability for the step - see [source files](#source) |
| `script`        | String               | The actual content of the script to run inside the Docker container created for the step |
| `ports`         | Sequence of mappings | Defines the ports for the service - see [ports](#ports) |
| `readiness`     | Mapping              | The readiness check for the service - see [service readiness](#readiness) |
| `environment`   | Sequence of mappings | Environment variables to make available to the step's container - see [environment variables](#environment) |
| `volumes`       | Sequence of mappings | Volumes to mount on the step container - see [volumes](#volumes) |
| `dockerfile`    | String               | Path to a Dockerfile to use for the step - see [using Dockerfiles](#dockerfiles) |

## Ports [](#ports)

If a service wants to open ports on the Docker container for that step, those ports will normally not be accessible. Services must explicitly choose to expose the ports started by that step, and allow them to be accessed. The `ports` property can be used to expose ports so that other services and steps in your workflow can access them - each port can be configured with the following keys:

| Key         | Type    | Description                              |
| ----------- | ------- | ---------------------------------------- |
| `name`      | String  | Name for the port - this should be a DNS label, and can be resolved using DNS by other services and workflow steps |
| `container` | Integer | The port on the container to expose      |
| `internal`  | Integer | Port at which other services and workflow steps can access the service (defaults to the same port as `container`) |
| `external`  | Integer | Port on the virtual machine host to map the `container` port to - you must specify a value in the range  `30000` to`32767` to expose the port externally |

Note that ports should be given a name with the `name` key in order to allow service ports to be discovered. Within the other steps of your workflow, and from other services, the name given to a port will resolve (using DNS resolution) to the service containing the port.

By default, if only `container` is specified, the port will be accessible by other services and steps in the workflow at the same port number. The `internal` key can be used to expose a container port as a different port to other steps in the workflow.

 In the example below, a lookup for `mysql` from the application that runs in the `Launch database` step will resolve to the IP of the database service. So, port `3306` on the container started for the `Launch database` step can be accessed by connecting to `mysql:4306` from other services and steps:

```yaml
steps:
 - service:
     name: Launch database
     image: mysql:8
     ports:
      - name: mysql
        container: 3306
        internal: 4306
- service:
    name: Build & run application
    image: maven:latest
    script: |-
      mvn spring-boot:run
```

Exposing a port externally will allow you to connect to that port through a browser - for example:

```yaml
steps:
 - service:
     name: Launch application
     image: node:latest
     script: |-
       npm start
     ports:
      - container: 8080
        external: 30080
```

This will expose the port on the Sandbox VM so that you can access it from a browser, using the IP of the Sandbox VM (see [Cluster status and management](http://localhost:8080/#!/docs/cli#management) to find out how to get the IP of the VM).

## Service readiness [](#readiness)

When workflows start services (see [services](#services)), Sandbox waits for the services to be ready before continuing on to the next step. In order to determine the readiness of a service, Sandbox uses a health check specified in the `readiness` key. *Exactly one* of the following keys should be defined for `readiness`, with each corresponding to a different type of health check:

| Key      | Type    | Description                              |
| -------- | ------- | ---------------------------------------- |
| `script` | Mapping | A script-based health check, which considers a 0 response returned by a script to be healthy |
| `http`   | Mapping | A HTTP health check, which considers a HTTP 2xx response to be healthy |
| `https`  | Mapping | A HTTPS health check, which considers a HTTP 2xx response to be healthy |
| `tcp`    | Mapping | A TCP health check, which considers a successful TCP connection being established to be healthy |

Consider an example which shows these different types of checks being used:

```yaml
steps:
 - service:
     name: Launch database
     image: mysql:8
     readiness:
       tcp:
         port: 3306
 - service:
     name: Build & run frontend
     image: node:8.4.0
     readiness:
       https:
         port: 8080
         path: /
         headers:
          - name: Accept
            value: text/plain
     script: |-
       npm install
       npm start
 - service:
     name: Build & run backend
     image: maven:latest
     readiness:
       script:
         path: backend-health.sh
     script: |-
       mvn spring-boot:run
```

In this example, Sandbox will wait at the `Launch database` step till a successful TCP connection can be made to port `3306`, which is the default MySQL port. Similarly, Sandbox will wait at the `Build & run frontend` step till a HTTP 2xx response is returned when making a HTTP GET request to the path `/` at port `8080`, with the specified request headers. Finally, Sandbox will wait at the `Build & run backend` step till the `backend-health.sh` script within the project directory returns an exit code of `0`.

As you can see, the following keys are available to configure the different types of health checks:

For TCP checks:

| Key    | Type    | Description                              |
| ------ | ------- | ---------------------------------------- |
| `port` | Integer | TCP port to attempt a connection to in order to establish healthiness |

For script checks:

| Key    | Type   | Description                              |
| ------ | ------ | ---------------------------------------- |
| `path` | String | Path (relative to project directory) to the script which performs the health check |

For HTTP and HTTPS checks:

| Key       | Type                                | Description                              |
| --------- | ----------------------------------- | ---------------------------------------- |
| `path`    | String                              | The path for the HTTP GET request        |
| `headers` | Sequence of `name`-`value` mappings | Headers for the HTTP GET request         |
| `port`    | Integer                             | Port to which an HTTP connection is made to send a HTTP GET request as part of the health check |

All health check types will be performed once every 10 seconds by default. The following additional keys can be used on any of the health check types to modify how the health checks behave:

| Key        | Type    | Description                              |
| ---------- | ------- | ---------------------------------------- |
| `interval` | Integer | The health check will be run every `interval` seconds (defaults to `10`) |
| `timeout`  | Integer | If a single run of the health check takes longer than `timeout` seconds, the health check is considered to have failed (defaults to `10`) |
| `retries`  | Integer | Only after a health check fails `retries` consecutive times is the service considered to be unhealthy by Sandbox (defaults to `3`) |
| `grace`    | Integer | A grace period in seconds during which any failing health checks will not count toward the retry count used in determining service health (defaults to `0`) |
| `skipWait` | Boolean | Whether to skip the wait for a service to be ready (defaults to `false`) |

In some circumstances, it may not make sense to immediately wait for a service to be ready before continuing. In these cases, you can set the `skipWait` property of the `readiness` check to `true` to tell Sandbox to continue past the step even if the service is not ready. Sandbox will continue until it reaches a compound step (see [Compound steps](http://localhost:8080/#!/#compound)) boundary, where it will wait for any services that are contained within the compound step to be ready.

## Service health [](#health)

Workflow services (see [services](#services)) are typically long-running, and run in the background, available for other workflow steps to use them as necessary. After services are started, and become ready, Sandbox can continue to monitor the health of services, making sure they are continuously available for other workflow steps to use. In order to tell Sandbox to monitor the health of services, use the `health` key. The `health` key defines a health check that has the same structure as the `readiness` check (see [service readiness](#readiness)). Consider the following example:

```yaml
steps:
 - service:
     name: Build & run frontend
     image: node:8.4.0
     readiness:
       http:
         port: 8080
         path: /
         headers:
          - name: Accept
            value: text/plain
     health:
       http:
         port: 8080
         path: /
         headers:
          - name: Accept
            value: text/plain
```

As you can see, the `health` check has the same structure as the `readiness` check. The difference is in how Sandox uses the checks: while the `readiness` check is used to initially determine when a service is ready, the `health` check is used to ensure that a long-running service continues to be healthy. If the service becomes unhealthy, Sandbox will automatically restart the service so that it continues to be available.

## Step image [](#image)

When running a workflow, Sandbox first builds an image for each `run`, `service`, and `generator` step. The base for that image can be specified using either the `image` or the `step` key. Note that for `run`, `service`, and `generator` steps, one of the two keys *must* be used to specify the base, unless a Dockerfile is defined for the step using the `dockerfile` key (see [using Dockerfiles](#dockerfiles)).

If you specify a Docker image with the `image` key, you can use the `<repository>:<tag>` format to specify any public image available on [Docker Hub](https://hub.docker.com/explore/). We recommend that you use official Docker images where possible.

If you specify the name of a previous step using the `step` key, note that that previous step must have already completed before it's final state can be used as an image. Sandbox will [commit](https://docs.docker.com/engine/reference/commandline/commit/) the previous step container's ending state as a new image, and use that image as the base for your step. An example of where you might choose to do this is if you compile and build binaries for your application in one step, and then want to use those binaries to run your application in a subsequent step. Here is an example of using `step` as the base image:

```yaml
steps:
 - run:
     name: Compile application
     image: maven:latest
     script: |-
       mvn clean install
 - service:
     name: Run application
     step: Compile application
     script: |-
       mvn spring-boot:run
```

When running this example, after the Docker container for the `Compile application` step finishes, Sandbox will commit the container's final state as an image. This image is then used as the base image for the Docker container for the `Run application` step.

## Source files [](#source)

When building the image for a `run`, `service`, or `generator` step, Sandbox starts with a base image (see [step image](#image)). On top of that base, Sandbox adds all source files within the workflow's project directory to the image for the step. The keys defined in `source` can be used to customize how source files are added to the image:

| Key            | Type                | Description                              |
| -------------- | ------------------- | ---------------------------------------- |
| `omit`         | Boolean             | Whether to omit all source files, so that no source files are added to the image built for the step (defaults to `false`) |
| `location`     | String              | Set the location where source files are added to the step image (defaults to `/app`) |
| `dockerignore` | String              | A path (relative to the project directory) to a file which contains rules for source files to ignore (defaults to `.dockerignore`) |
| `include`      | Sequence of strings | A list of file patterns that specify files to include in the step image |
| `exclude`      | Sequence of strings | A list of file patterns that specify files to exclude from the step image |

Note that the file specified by the `dockerignore` key should contain ignore rules in the format specified by  [Docker's documentation on .dockerignore files](https://docs.docker.com/engine/reference/builder/#dockerignore-file). That means that an existing `.dockerignore` file should be honored when building a step image. However, the `dockerignore` key allows you to use a different file for each step.

The syntax for the `include` and `exclude` patterns are the same as [Go's `filepath.Match` patterns](https://golang.org/pkg/path/filepath/#Match). If you use these properties, only the source files that match the include patterns are selected first for inclusion. From this set, any files matched by any of the exclude patterns are excluded to arrive at the final list of files to add to the step image.

The simple example below includes shows the use of `include` to add only the `package.json` and `package-lock.json`files to the step image:

```yaml
steps:
 - run:
     image: node:8.4.0
     source:
       include:
        - package.json
        - package-lock.json
     script: |-
       npm install
       npm run build
```



If you use the `dockerignore` key for a particular step, you cannot also use the `include` and `exclude` keys for the same step. You *must* use one or the other.

It's important to note that because source files are added to the image of the container used for the step, changes made to these files within the step container only affect the container's file system. That means changes to these files will not be reflected to the actual source files in your project directory. In order to make changes to your project files from within a step container, you will need to use [volumes](#volumes).

## Caching step images [](#cache)

Normally, when you use the ending state of a previous step as the image for a subsequent step (see [step image](#image)), the previous step will run every single time, and a fresh image is generated every single time. You may want to change this behavior to cache the image generated for that previous step, if the input source files to the step have not changed. You can use the `cache` property on a step to tell Sandbox to cache the image that is generated for that step. The cached image will be used while the input source files (defined using the keys descripted in [source files](#source)) to the step do not change. While the cached image is still valid, subsequent runs of the workflow will not re-run the step on which `cache` is set to `true`.

This is useful for long-running setup and installation steps that do not change often, such as in this example:

```yaml
steps:
 - run:
     name: Install dependencies
     image: node:latest
     cache: true
     script: |-
       npm install
 - run:
     name: Run frontend
     step: Install dependencies
     script: |-
       npm start
 - service:
     name: Build & run backend
     image: maven:latest
     script: |-
       mvn spring-boot:run
```

## Environment variables [](#environment)

Environment variables can be set on the Docker container that is run for a particular step using the `environment` key. The value for `environment` should be a sequence of mappings, where each mapping can be *either* a `name`-`value` pair, or a `file` entry:

| Key     | Type   | Description                              |
| ------- | ------ | ---------------------------------------- |
| `name`  | String | Name of environment variable to set, when specifying a `name`-`value` pair |
| `value` | String | Value for environment variable, when specified as a `name`-`value` pair |
| `file`  | String | Path to a file containing environment variable definitions relative to the project directory |

When using the `file` key, Sandbox expects a file that uses the syntax described [here](https://docs.docker.com/compose/env-file/).

Here is a simple example which shows environment variables being used:

```yaml
steps:
 - run:
     environment:
      - name: MYSQL_USER
        value: user
      - name: MYSQL_PASSWORD
        value: pass
      - file: app-server-props.env
```

As the example shows, `file` entries can be mixed together with `name`-`value` pairs.

## Volumes [](#volumes)

By default, files in the workflow's project directory are added to the step image as described in [source files](#source). Among other things, this means any changes made to these files from within the container won't be reflected on the files in the project directory on the host. If a directory is mounted as a volume instead, changes made to the files from within the container will be reflected to the files on your host as well. Each volume is configured with the following keys:

| Key                    | Type   | Description                              |
| ---------------------- | ------ | ---------------------------------------- |
| `name`                 | String | A name for the volume                    |
| `mountPath` (Required) | String | The path within the step container where the volume will be mounted |
| `hostPath`             | String | Path on the host to mount as a volume    |

Here is an example where each step mounts build output directories as volumes:

```yaml
steps:
 - run:
     name: Build backend
     image: maven:latest
     volumes:
      - mountPath: /app/target
        hostPath: ./target
     script: |-
       mvn clean install
 - run:
     name: Build frontend
     image: node:latest
     volumes:
      - mountPath: /app/dist
        hostPath: ./dist
     script: |-
       npm run build
```

You can see that the `target` directory is mounted as a volume for the `Build backend` step. The output that Maven generates (which typically goes into a `target` directory) will be generated into the volume, mounted at `/app/target`. The effect will be that the Maven generated output (in this case, the application binary JAR) will be available in the `target` directory on the host.

Similarly, the output generated into the `dist` directory by the `Build frontend` step will be available on the `dist` directory on the host.

You can see that the `hostPath` in both steps were relative paths. These paths are relative to the project directory. It is highly recommended that the `hostPath` you use for volumes is a relative path, because this will result in the most portable workflows. Sandbox will allow you to use absolute paths for the `hostPath`, which may be useful in some limited circumstances. For security reasons, Sandbox will only allow absolute paths within the user's home directory.

**Warning:** Make sure to only run workflows that you trust! Malicious workflows can mount absolute paths as volumes and modify files in your home directory without your knowledge.

## Error handling [](#errors)

Normally, if a particular workflow step fails, Sandbox will fail the entire workflow. You can tell Sandbox to ignore certain types of failures by using the `ignoreErrors` key. The value for the `ignoreErrors` key is a mapping, which can define three keys:

| Key          | Type    | Description                              |
| ------------ | ------- | ---------------------------------------- |
| `missing`    | Boolean | Whether to ignore variable placeholders that cannot be resolved because they refer to a missing variable (defaults to `false`) |
| `validation` | Boolean | Whether to ignore any errors arising from the validation of step definitions - for example, incorrect types (defaults to `false`) |
| `failure`    | Boolean | Whether to ignore failures that occur during the building of a step image, or the running of a step (defaults to `false`) |

You can define this key both at the workflow level, and at an individual step level. When defined at the workflow level, the error handling defined there applies globally to all steps in the workflow, except those which override the global behavior with their own error handling.

The following example shows how `ignoreErrors` can be used to ignore validation errors:

```yaml
steps:
 - service:
     name: Run frontend
     image: node:latest
     ignoreErrors:
       validation: true
     script: |-
       npm run watch
     ports:
      - container: ${httpPort}
```

Normally, Sandbox will fail a step if it fails to pass validation after all variables in the step have been expanded. In this example, consider the case when variable `httpPort` expands to a non-integer value (only integers are valid). Normally, Sandbox would fail the workflow. However, because `ignoreErrors` is specified using `validation` set to `true`, Sandbox would continue running the workflow.

## Parallel steps [](#parallel)

By default, `run`, `external`, and `generator` steps are sequential. That means that Sandbox will wait for the Docker container for the step to complete before it proceeds to the next step. You can set the `parallel` key `true` to tell Sandbox not to wait till the step completes before proceeding.

For example:

```yaml
steps:
 - run:
     name: Build frontend
     image: node:latest
     parallel: true
     script: |-
       npm install
       npm run build
 - run:
     name: Build backend
     image: maven:latest
     script: |-
       mvn install
```

Sandbox does not wait for the `Build frontend` step to finish before it starts running the `Build backend` step. Since the two are independent builds, they can be run in parallel.

## Compound steps [](#compound)

[Parallel steps](#parallel) introduce a new challenge for workflows. There may be cases where several steps can be done in parallel but a subsequent step needs to wait for one or more of those parallel steps to complete before running. In order to wait for parallel steps, you can use compound steps, and specify sub-steps using the `steps` key:

```yaml
steps:
 - compound:
     name: Build application
     steps:
      - run:
          name: Build frontend
          image: node:latest
          parallel: true
          script: |-
            npm install
            npm run build
      - run: Build backend
          image: maven:latest
          parallel: true
          script: |-
            mvn install
  - run:
      name: Run tests
      image: ubuntu:latest
      script: |-
        run-tests.sh
```

Sandbox waits for any and all parallel steps within a compound step to complete before it continues past the compound step. So in the above example, Sandbox waits for the `Build frontend` step and the `Build backend` step to finish before running the `Run tests` step. Note that within the compound step, both steps are executed in parallel.

It should be noted that any sequential steps within compound steps run sequentially as you would expect them to. [Services](#services) within compound steps wait till they are ready as you would expect them to.

## Calling external workflows [](#external)

Workflows can be composed together by having a step that calls another workflow - the `target` property is used to specify the name of a workflow to call:

```
steps:
- name: Build application
  target: Build app
- name: Run application
  target: Run app
```

This simple workflow is composed of calls to two other workflows. The first step calls the `Build app` workflow. Note that this will run the workflow in the `Build app.wflow` file in the project's `workflows` directory. After the `Build app` workflow finishes, the control returns to this workflow, and the next step is run. In this case, the next step is to run the `Run app` workflow.

When calling workflows, all the workflow variables from the current workflow are made available to the workflow being called. Any initial variables the workflow being called defines will be overwritten. In some cases, you may want to preserve the initial state of variables set by the workflow being called. You can preserve these variables by using the `excludeVariables` property:

```
variables:
- httpPort: 8080
steps:
- name: Run application
  type: call
  target: Run app
  excludeVariables:
    - httpPort
    - mysqlPort
```

In this simple example, the `httpPort` and `mysqlPort` variables will not be passed to the `Run app` workflow that is being called.

When a workflow that is called finishes, the values of all variables in the workflow that was called are copied back to the calling workflow. Again, the `excludeVariables` property can be used to exclude variables.

## Running generated workflows

Sandbox also allows you to specify a step that generates a workflow, and then run the generated workflow. This is done using the `generator` property, which is used instead of the `script` property (see [Step script](http://localhost:8080/#!/#script)).

```
steps:
- name: Generate and run workflow
  generator: workflow-generator.sh
  environment:
    - name: HTTP_PORT
      value: 8080
```

In this example, the `workflow-generator.sh` script within the project source directory is expected to generate a workflow, which will then be run by Sandbox. Note that most of the other properties available in a step that executes a script are available to steps that generate workflows. In this example, the `environment` property is used to set environment variables on the Docker container in which the generator script is run.

Sandbox expects the generator script to write the generated workflow YAML to stdout. The generated YAML should be preceeded with a line that has the text `workflow {` and followed by a line with the text `}`. Sandbox will consider all lines in between those marking lines as the YAML content of the workflow.

The rules for variables that apply to calling other workflows (see [Calling other workflows](http://localhost:8080/#!/#calling)) also apply when Sandbox runs a generated workflow. In particular, the `excludeVariables` can be used to exclude variables copied between the workflow boundaries.

## Using Dockerfiles [](#dockerfiles)

In certain scenarios, you may want to use your own Dockerfile for a step. Sandbox allows you to do so using the `dockerfile` property. Note that if you define a step using `dockerfile`, you cannot use the `image`, `step`, or`script` properties for that step. All other keys normally available for that step type are still available.

Here is a simple example of a step that uses the `build-Dockerfile` (note that paths are relative to the project directory):

```yaml
steps:
 - run:
     name: Build & run application
     dockerfile: build-Dockerfile
```