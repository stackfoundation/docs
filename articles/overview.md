## What is Sandbox? [](#what)

**StackFoundation Sandbox** is a free tool that allows you to _fully reproduce_ complex development tasks using Docker-based workflows. You can design a workflow that reproduces the development tasks that you perform regularly. This can include:

*   Building your application
*   Running unit tests, integration tests, functional tests, or any other automated tests
*   Running your application in order to debug it, along with all of it's third-party dependencies (databases, etc.)
*   Deploying your application somewhere

Sandbox runs workflows that reproduce tasks that are multi-step, multi-language, and multi-platform.

## Why should I use Sandbox? [](#why)

* **Reproduce complex tasks** - Sandbox runs workflows where each workflow step can run a Docker container. Because each step runs a Docker container, you 
* **Multi-language, multi-platform** - Because each workflow step runs a Docker container, you can have each step run different languages, or platforms
* **Full reproduciblity** - On a new machine, Sandbox installs everything necessary to run workflows, even Docker. That means you be sure that running a workflow fully reproduces

## How do I use Sandbox? [](#use)

* **Create workflows** - Workflows are YAML files that describe a sequence of steps, where each step can run a script within a Docker container. You create workflow `.yml` files within a `workflows` folder in your project.
* **Add Sandbox to your project** - There is no installation with Sandbox - you simply [download](/downloads) and unzip a small zip file (&lt;200KB) into your project directory. This provides all the necessary binaries to run Sandbox on your project on Linux, macOS and Windows.
* **Run workflows** - Once you have Sandbox in your project, you can run the workflows you created by issuing a `sbox run` command.
* **Commit Sandbox and workflows to Git** - Committing Sandbox and your workflows into your Git repo means anyone can run your workflows immediately. The small size of the binaries means you can do so without being concerned about the impact on your repo size.

## How does Sandbox work? [](#how)


## How do I get started? [](#start)

To get started with Sandbox, see [Getting Started guide](/docs/getting-started), or jump into our language-specific tutorials:
