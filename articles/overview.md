## What is Sandbox? [](#what)

**StackFoundation Sandbox** is a free tool that allows you to fully reproduce complex development tasks using Docker-based workflows that are multi-step, multi-language, and multi-platform. You can design a workflow that reproduces the development tasks that you perform regularly. This can include:

*   Building your application
*   Running unit tests, integration tests, functional tests, or any other automated tests
*   Running your application in order to debug it, along with all of it's third-party dependencies (databases, etc.)
*   Deploying your application somewhere

## Why should I use Sandbox? [](#why)

* **Reproduce complex tasks** - Sandbox runs workflows where each workflow step can run a Docker container. Docker-based workflows allow you to create powerful multi-step, multi-language, and multi-platform workflows to reproduce your most complex tasks.
* **Multi-language, multi-platform** - Because each workflow step runs a Docker container, you can have each step run different languages, or platforms
* **Full reproduciblity** - On a new machine, Sandbox installs everything necessary to run workflows, even Docker. That means you be sure that running a workflow fully reproduces
* **Available upon checkout** - Sandbox is tiny (&lt;200KB) by design so that it can be checked in to your Git repo together with your project, and your workflows. That allows anyone that checks out your project repo to be able to perform all development tasks immediately upon checkout. Because Sandbox reproduces everything, including the environment needed to run workflows, anyone that checks out your project has everything they need to work with it, without installing any other software.

## How do I use Sandbox? [](#use)

* **Create workflows** - Workflows are YAML files that describe a sequence of steps, where each step can run a script within a Docker container. You create workflow `.yml` files within a `workflows` folder in your project.
* **Add Sandbox to your project** - There is no installation with Sandbox - you simply [download](/downloads) and unzip a small zip file (&lt;200KB) into your project directory. This provides all the necessary binaries to run Sandbox on your project on Linux, macOS and Windows.
* **Run workflows** - Once you have Sandbox in your project, you can run the workflows you created by issuing a `sbox run` command.
* **Commit Sandbox and workflows to Git** - Committing Sandbox and your workflows into your Git repo means anyone can run your workflows immediately. The small size of the binaries means you can do so without being concerned about the impact on your repo size.

## How does Sandbox work? [](#how)


## How do I get started? [](#start)

To get started with Sandbox, see [Getting Started](/docs/getting-started), or jump straight into our language-specific quick start guides:

* Javascript
  - [Build and run Node.js apps](/docs/nodejs)
  - [Hot reload with webpack](/docs/webpack)
* Java
  - [Build and run apps with Maven](/docs/maven)