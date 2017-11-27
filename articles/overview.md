# Overview

## What is Sandbox? [](#what)

**StackFoundation Sandbox** is an open source tool that allows you to consistently reproduce complex development tasks using Docker-based workflows that are multi-step, multi-language, and multi-platform. You can design a workflow that reproduces the development tasks that you perform regularly. This can include:

*   Building your application
*   Running unit tests, integration tests, functional tests, or any other automated tests
*   Running your application, along with all of it's third-party dependencies (databases, etc.)
*   Deploying your application somewhere

Together with [StackFoundation Cloud](/cloud), it provides consistently reproducible development from local to CI to production.

## Why should I use Sandbox? [](#why)

* **Reproduce complex tasks** - Sandbox runs workflows where each workflow step can run a Docker container. Docker-based workflows allow you to create powerful multi-step, multi-language, and multi-platform workflows to reproduce even your most complex tasks.
* **Multi-language, multi-platform** - Because each workflow step runs a Docker container, you can have each step run a different language and platform.
* **Consistent reproduciblity** - On a new machine, Sandbox installs everything necessary to run workflows, even Docker. That means that in addition to ensuring your workflow is repeatable, Sandbox ensures that the environment needed to run your workflow is itself reproducible consistently.
* **Effortless out-of-the-box** - Sandbox is tiny (&lt;200KB) by design so that it can be committed to your Git repo, together with your project, and your workflows. That allows anyone that checks out your project repo to be able to perform all development tasks immediately upon checkout. Because Sandbox reproduces everything, including the environment needed to run workflows, anyone that checks out your project has everything they need to immediately run workflows, without installing any other software.

## How do I use Sandbox? [](#use)

* **Install Sandbox** by [downloading](/downloads) and unzipping a small zip file (&lt;100KB) into your project directory. This provides all the necessary binaries to run Sandbox for your project on Linux, macOS and Windows. Take a look at [Installing Sandbox](/docs/installing) for more information.
* **Create workflows,** which are YAML files that describe a sequence of steps, where each step can run a script within a Docker container. You create workflow `.yml` files within a `workflows` folder in your project.
* **Run workflows** by issuing a `sbox run` command, once you have Sandbox in your project. No other software is necessary.

## How does Sandbox work? [](#how)

* **Bootstrapping** - When you run Sandbox for the first time on a machine, the tiny Sandbox binaries perform a bootstrap step in which they download additional components, and setup a single-node Kubernetes cluster on the machine. See [Sandbox bootstrapping](/docs/bootstrapping) for details.
* **Running workflows** - After a single-node Kubernetes cluster is bootstrapped, Sandbox uses the Kubernetes cluster to run the workflows that are requested. Workflow steps are executed by translating them into Kubernetes pods, services, and other constructs.
* **Building images** - During workflow execution, any images that need to be built for the various worfklow steps are built by Sandbox by talking directly to the Docker daemon started in the single-node Kubernetes cluster. That means you don't need pre-built images to use them in workflows.

Of course, you can get full access to both the Docker daemon and Kubernetes cluster started by Sandbox, and control the lifecycle of both.

## How do I contribute? [](#contribute)

Sandbox is an open source project, and contributions in the form of bug reports, feature requests, and code are very welcome. If you are interested in contributing, head to the [Sandbox GitHub repo](https://github.com/stackfoundation/sandbox).

## How do I get started? [](#start)

To get started with Sandbox, see [Getting started](/docs/getting-started), or jump straight into our language-specific quick start guides:

* Javascript
  - [Build and run Node.js apps](/docs/nodejs)
  - [Hot reload with webpack](/docs/webpack)
* Java
  - [Build and run apps with Maven](/docs/maven)