# Workflows

Sandbox runs **workflows** that are comprised of a series of steps, where each step runs a script within a Docker container. Because scripts are run within a Docker container, they have a predictable and well-known environment in which to run, and thus run consistently. Workflows allow you to consistently reproduce all your development tasks.

## Workflow files

Workflows are specified as YAML files, with a `.yml` extension, and generally stored within a folder called `workflows`, within a project.

Workflows can be manually edited using any text editor (since they are just YAML files). Workflows are run using the Sandbox CLI.

Workflow files have the following top level keys:

| Key            | Description                              |
| -------------- | ---------------------------------------- |
| `steps`        | Defines a list of steps that make up the workflow |
| `ignoreErrors` | Defines how errors are handled while running the workflow - see [error handling](#errors) for details |
| `variables`    | Defines global variables used in the workflow |