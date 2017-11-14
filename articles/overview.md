## What is Sandbox? [](#sandbox)

**StackFoundation Sandbox** is a tool that allows you to run workflows that{' '} _reliably_{' '} automate your most common development tasks. You can design a workflow that automates any development task that you perform regularly. This can include:

*   Building your application
*   Running unit tests, integration tests, functional tests, or any other automated tests
*   Running your application in order to debug it, along with all of it's third-party dependencies (databases, etc.)
*   Deploying your application somewhere

## Making your existing scripts more reliable [](#reuse)

You may already have scripts that automate some or all of your most common development tasks. These scripts are probably written in various languages, and they probably need a particular environment in order to run. For example, you may have:

*   Bash scripts that require having a Linux distro with Bash installed such as Ubuntu
*   Node.js scripts that require having a particular version of Node.js installed
*   Maven POMs that require having a particular version of Maven, as well as a JDK
*   Python scripts that require a particular version of Python

If you don't have the environment that's required by a particular script, you have to first setup the environment before you can run the script. With Sandbox, you can ensure that these scripts have a well-known, predictable environment available to them. That's because Sandbox will run your scripts within a Docker container. And you can guarantee that these Docker containers have everything your scripts need in order to run. So, taking the examples discussed above, you can ensure:

*   Bash scripts run within a [Ubuntu Linux](https://hub.docker.com/_/ubuntu/) container
*   Node.js scripts run within a [Node.js](https://hub.docker.com/_/node/) container
*   Maven scripts run within a [Maven](https://hub.docker.com/_/maven/) container
*   Python scripts run within a [Python](https://hub.docker.com/_/python/) container

Running your scripts in a well-known, predictable environment means they run more reliably.

## Docker-based workflows [](#workflows)

At the heart of Sandbox are workflows, YAML files that are made up of a sequence of steps. Each step in a workflow specifies a script to run within a Docker container. Each step runs in a separate Docker container, and each container can be created from a different base image. So for example, you can define a workflow where:

*   You run a bash script to build your application within a{' '} [Ubuntu Linux](https://hub.docker.com/_/ubuntu/) container
*   In the subsequent step, run a Node.js script that runs your functional tests within a{' '} [Node.js](https://hub.docker.com/_/node/) container

Workflows are the way Sandbox allows you to run your scripts more reliably. Workflows are described in detail in the {' '} <link to="/docs/workflows">Workflows section of the documentation.

## Run workflows straight from your Git repo [](#repo)

Workflows are run using the [Sandbox CLI (Command Line Interface)](/docs/cli). For example, running a workflow called `functional-tests` is done using the command{' '} `sbox run functional-tests`.

When you run a workflow, the Sandbox CLI takes care of setting up all the software necessary to run your workflows. This includes Docker itself, and any images your workflows use. That means when using the CLI, you don't need any other software - not even Docker!

The `sbox` command is actually a set of tiny cross-platform binaries. Two executable scripts, `sbox` and `sbox.bat` take care of running the correct binary for your platform. Altogether, the Sandbox CLI scripts and binaries for all supported platforms (including Mac OS X, Windows and many Linux distros) are less than 200KB.

The small size of the binaries is intentional: we wanted to make them small enough to check into your source control repository. If the binaries are checked into your project's repo, it means that anyone that checks out your project's source code repo can immediately run workflows using Sandbox. Imagine being able to check out your project, and run a workflow that immediately runs your app's functional tests:

```bash
git checkout https://github.com/rchodava/workflow-designer
cd workflow-designer
./sbox run functional-tests
```

Checking in the Sandbox CLI into your repo means anyone can run your workflows immediately. The small size of the binaries means you can do so without being concerned about the impact on your repo size. And because the binaries support a variety of platforms, your workflows (and the scripts they run) are incredibly portable.

## Architecture [](#architecture)

In order to allow you to run workflows on any machine with just a few small binaries, the Sandbox CLI does a few things behind the scenes. Let's examine what happens when you run a workflow using a command like `sbox run functional-tests`:

*   The `sbox` command is actually an executable Unix shell script (on Windows, it's the `sbox.cmd` command script) which runs one of several cross-platform "wrapper" binaries found in the `sbox-cli` folder. For example, on Mac OS X, `sbox` executes a "wrapper" binary called `wrapper-x.y.z-darwin-386` that is about 50KB in size.
*   The wrapper binaries download additional components for the CLI that make up the core of the CLI. These components consist of two binaries called "bootstrap", and "core", which are downloaded to a platform-specific location (on Mac and Linux this is usually `/usr/local/sf`, and on Windows, it's within the `Roaming/sf` folder inside the user's `AppData` folder). This is done only when the CLI is run for the first time. Subsequent calls will generally use the core components that were previously downloaded. Occasionally, updated versions of the CLI may be downloaded if present. By default, the CLI components are downloaded from the official StackFoundation servers. If necessary, the CLI can be told to download these components from alternate mirror servers that you setup. You can see how this is done in the section: [{translate('TITLE_CLI_MIRRORS')}](/docs/cli#mirrors).
*   After the core CLI components are downloaded, this CLI core then sets up a virtual machine on your operating system. The exact mechanism by which a virtual machine is created depends on your operating system. Native support for virtualization is used where available - for example, the Hypervisor framework is used on Mac OS X, and HyperV is used on versions of Windows where it is present. When native virtualization support is not available, the CLI will install a version of [VirtualBox](https://www.virtualbox.org/) for your platform. The CLI core creates the virtual machine from an image which contains a lightweight Linux distribution, and which runs a Docker daemon.
*   After the virtual machine is created, the CLI core then sets up a single-node [Kubernetes](https://kubernetes.io/) cluster.

We refer to the above process as "bootstrapping" Sandbox. Bootstrapping generally occurs only on first run on a machine, and when there are updates to Sandbox. After Sandbox has been bootstrapped, and the Kubernetes cluster is up and running, Sandbox will run workflows using Kubernetes as the target platform. That means it translates workflow concepts into Kubernetes concepts - for example, a step execution is performed as the execution of a Kubernetes pod.