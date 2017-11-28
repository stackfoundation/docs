## CLI (Command Line Interface)

The Sandbox CLI (Command Line Interface) is the tool used to run workflows. If you have the CLI, simply run `sbox` to run the CLI. With no flags and arguments, the CLI will show you help on all the available commands.

## Running workflows (`run`) [](#run)

In order to run a workflow, simply invoke the CLI with the run command at the project root:

```
sbox run functional-tests
```

The name of the workflow is the first argument passed after the `run` command. Any arguments passed after the workflow name are passed into the workflow as an array of positional arguments:

```
sbox run functional-tests arg0 arg1 arg2 [...]
```

See [the section on workflows](/docs/workflows#cli-parameters) to know how to define and use these arguments in your workflow.

## Initializing a project (`init`)[](#init)

In order to run the `sbox` command, you will need to [download the Sandbox CLI](/downloads) and add it to your project by extracting the zip file containing cross-platform binaries and scripts. That will allow you to run workflows contained within that project. Note that when you extract the zip file into your project's directory, you will see the following files are present:

- `project1`
  - `sbox-cli`
    - Sandbox cross-platform binaries...
  - `sbox`
  - `sbox.bat`
  - Other project files...

Note that several cross-platform binaries are present within a `sbox-cli` directory, inside the directory you extracted the zip. These include binaries for Windows, Mac OS X, and Linux. At the project root, a Unix shell script called `sbox`, and a Windows batch script called `sbox.bat` have been extracted. Invoking these scripts will run the CLI (picking the right binary for the operating system on which the scripts are run).

The files that make up the CLI are tiny: all together, they are less than 200KB! These files are designed to be committed to your source repository. This will allow anyone who checks out your source repository to immediately run the CLI directly within your project. Take a look at the section called [Run workflows straight from your Git repo](/docs/overview#repo) for more.

Once you have obtained the CLI, and have it available globally (which can be done by running `sbox install`), you can initialize a project simply by running the `init` command, which is invoked at the project root:

```
sbox init
```

Running `init` will create the same structure described above in the directory in which it is run - that is, it will copy the cross-platform binaries and scripts for running the CLI.

## Project layout [](#layout)

The CLI is typically run at the project root. The CLI expects a project using workflows to have a specific simple layout. There should be a directory called `workflows` that contains all the workflows for the project. Consider a sample project layout:

- `project1`
  - `workflows`
    - `Build application.wflow`
    - `Run application.wflow`
    - `Unit tests.wflow`
    - `Functional tests.wflow`
  - Other project files...

In this example, the CLI should be run in the project root directory - `project1`. This project has 4 workflows, named "Build application", "Run application", "Unit tests", and "Functional tests". Note that when referring to workflow names, we exclude the `.wflow` extension used for workflow files.

## Listing workflows (`list`) [](#list)

You can list the workflows in a project using:

```
sbox list
```

## Cluster status and management [](#management)

Workflows are run as Docker containers on a virtual machine (VM) running a single-node Kubernetes cluster. See the section on [Architecture](/docs/overview#architecture) for more. This VM and cluster are automatically created when the first workflow is run. The CLI allows you to view the status and manage the state of this VM and cluster using the following commands:

You can view the status of the VM and cluster, and the address of the Kubernetes dashboard (the IP in this address is the IP of the VM), using:

```
sbox status
```

You can get a listing of environment variables needed to connect to the Docker daemon running on the VM using (similar to a`docker-machine env` command, for those of you familiar with Docker Machine):

```
sbox docker-env
```

If you no longer need to run workflows, you can stop the VM and cluster using the following command:

```
sbox shutdown
```

Note that if you run workflows after stopping the VM and cluster, Sandbox will automatically restart them.

Finally, you can remove the VM and cluster created by Sandbox by using the following command:

```
sbox rm
```

We recommend using the `shutdown` command to just stop the VM and cluster instead of removing them altogether. Note that if you run workflows after removing the VM and cluster, Sandbox will automatically create new ones.

## Debugging sandbox (`--debug`)[](#debug)

If you find the need to debug the sandbox process itself, to track some issues that might be due to its inner workings, or suspected bugs, you can run any and all commands usimg the `debug` flag:

```
sbox [run/list/status/install...] --debug
```

## Using mirrors [](#mirrors)

Earlier in the section titled "[Initializing a project](/docs/cli#init)", we talked about the Sandbox CLI being made up of a set of tiny cross-platform binaries. As described in the [Architecture](/docs/overview#architecture) section of the overview, these binaries need to download additional CLI components in order to work. By default, these components are downloaded from the official StackFoundation servers. In some circumstances, you may want to change this behavior so that these CLI components are downloaded from alternate mirror servers (for example, mirror servers setup by your organization).

A mirror server is just a standard HTTP or HTTPS server that mirrors the following files:

- The `checksums.sha256` file (the folder on the mirror server in which this file exists establishes the URI that is considered the root of the mirror)
- Every file referred to in the `checksums.sha256` file, which should be available relative to the root of the mirror

You can setup a mirror by mirroring the content found on the official StackFoundation servers, starting with the file at`https://updates.stack.foundation/checksums.sha256`.

You can tell the Sandbox CLI to use mirrors to download CLI components by creating a file called `mirrors` inside the `sbox-cli`folder. This should just be a plain text file which lists the mirror server root URIs, one per line. For example, a file containing the lines `http://company.com/sbox-mirror` and `https://company.com/sbox-mirror` will look for the `checksums.sha256` file at`https://company.com/sbox-mirror/checksums.sha256`.

The "wrapper" binaries that perform the initial download of core CLI components are very small. In order to maintain their small size, they only support downloading from a HTTP server (TLS support increases the size of binaries significantly). This means that you must have at least one HTTP mirror. The "wrapper" binaries perform a SHA-256 checksum of the components they download (the checksums are hard-coded into the binaries) to ensure they are only executing known binaries.

After the initial download, further CLI components are downloaded from a HTTPS server. This rule is enforced: subsequent CLI components must be downloaded over HTTPS for security. This means that if you are using your own custom mirrors, you must have at least one HTTPS mirror. Together with the previous point, that means if you are using your own mirrors, you must have at least one HTTP mirror, and at least one HTTPS mirror.

Alternatively, URIs with a `file://` scheme can also be specified as mirrors, in order to retrieve CLI components from the file system. Again, a copy of the `checksums.sha256` file is expected to be present in the folder specified, and the various files referenced by that file must be present.