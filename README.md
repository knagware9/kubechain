# Kubechain
The Kubechain CLI has been created to ease the deployment of blockchains onto a Kubernetes cluster.
At the time of writing most blockchains do not support kubernetes deployments or the deployment process is arduous.

## Components
Kubechain consists of the following components.

- A CLI
- Blockchain Adapters
- A minimal Kubernetes SDK

### CLI
The CLI is responsible for handling user commands and calling the the corresponding BlockchainAdapter.
At the moment of writing the CLI's commands are tightly coupled with the adapters. 
I'm working towards decoupling the CLI and the commands BlockchainAdapters expose. However, this will take some time.

### Blockchain Adapters
A blockchain adapter provides an API for the creation and deletion of Blockchain and Kubernetes configuration.
At its core an adapter transforms Blockchain configuration to configuration for a Kubernetes cluster.

### Kubernetes SDK
The SDK provided by Kubechain is intended solely to ease the creation of Kubernetes configuration. 
As of right now the only output type it supports is JSON. YAML is not supported.

### Interaction
Interaction between the components is as follows.

- The CLI provides a UI for a user to interact with a BlockchainAdapter. 
- BlockchainAdapters in turn provide CommandExecutors for the CLI to call.
- A BlockchainAdapter 'may' use the Kubernetes SDK to create the necessary resources.

## Usage

### CLI
As of right now Kubechain is intented to be used via the CLI. Blockchain adapters should support at least the following commands:

- ``create chain-config <chain>``
  - Creates the blockchain configuration.
- ``create kubernetes-config [blockchain-target:b] [kubernetes-target:k]``
  - Creates the kubernetes resources.
- ``create config [blockchain-target:b] [kubernetes-target:k]``
  - Creates all the configuration necessary to create a Kubernetes resources.
- ``create cluster [blockchain-target:b] [kubernetes-target:k]``
  - Creates a Kubernetes cluster from previously created Kubernetes resources.
- ``delete chain-config <chain>``
  - Deletes existing blockchain configuration from the CLI host.
- ``delete kubernetes-config [blockchain-target:b] [kubernetes-target:k]``
  - Deletes existing Kubernetes resources from the CLI host.
- ``delete config [blockchain-target:b] [kubernetes-target:k]``
  - Deletes all existing configuration from the CLI host.
- ``delete cluster [blockchain-target:b] [kubernetes-target:k]``
  - Deletes the [running] Kubernetes cluster using previously created Kubernetes resources.

In the above ``<chain>`` is the name of the blockchain you wish to ``target``.
See the Targets section for a list of supported blockchain targets.

#### Targets

#### Blockchains
Currently supported blockchain targets for the ``chain`` and `[blockchain-target:b]` arguments are:

- ``fabric``, [Hyperledger Fabric](https://github.com/hyperledger/fabric/).
- ``burrow``, [Hyperledger Burrow](https://github.com/hyperledger/burrow/).

Hyperledger Burrow (``burrow``) support is in the works.

#### Kubernetes 
Currently supported Kubernetes targets for the `[kubernetes-target:k]` argument are:

- `minikube`, [Minikube](https://github.com/kubernetes/minikube/)
- `gce`, [Goocle Cloud Engine](https://cloud.google.com/)

#### Paths
Below you can find some useful information about the paths Kubechain creates or requires.

Running ``kubechain create config <chain>`` will result in the creation of several directories.
The directories:
- `<cwd>/blockchains/<chain>`
- `<cwd>/kubernetes/<chain>`

Currently any configuration for a blockchain needs to be added manually to:
- `<cwd>/configuration/<chain>`

##### Fabric Example
Below you find an example of a directory tree for Hyperledger Fabric and the corresponding Kubernetes configuration generated by Kubechain.

- `cwd`
  - configuration <- input for blockchains
    - fabric
      - configtx.yaml
      - crypto-config.yaml
  - blockchains <- input for Kubernetes
    - fabric
      - bin
        - configtxgen
        - cryptogen
      - crypto-config
        - ...
  - kubernetes <- final result
    - fabric

## Tutorials
For all available tutorials check out [the tutorial documentation](https://github.com/kubechain/kubechain/tree/master/docs/tutorials).

### Creating a Hyperledger Fabric cluster on Minikube

#### Pre-requisites
- You've installed NodeJS (version 8 and higher) and NPM.
- You've installed Minikube and added it to the PATH variable on your system.
- You've installed Kubectl and added it to the PATH variable on your system.

#### Setup the environment
1. Open a terminal/console.
1. Run: ``npm install -g kubechain``
1. Run: ``minikube start``

#### Hyperledger Fabric configuration
1. At a preferred ``hostpath`` create a new directory named ``configuration``.
1. Create sub-directory named ``fabric`` in the ``configuration`` directory.
1. Add the standard Hyperledger Fabric configuration files to the ``fabric`` directory.
   - These are ``configtx.yaml`` and ``crypto-config.yaml``.

#### Create the Kubernetes Cluster
1. Open a new terminal/console.
1. Change your current directory to the ``hostpath`` where you created the ``configuration`` directory.
   - i.e. Run: ``cd hostpath``, not `cd hostpath/configuration`
1. Run: ``kubechain create config -b fabric -k minikube``
1. Run: ``kubechain create cluster -b fabric -k minikube``
1. Verify that the cluster is operational by your preferred means.

# Support
As of right now I'm doing this project on my own as it is part of my Master Thesis.
Currently I do not have the time to elaborately answer questions about Kubechain nor provide full support.
I'll try to answer as quickly as I can but don't count on instant replies.

# Alternatives

## Helm
One problem with Kubechain is providing support for all possible Kubernetes targets.
This would require Kubechain to have fully configurable components.
This is exactly to what [Helm, Kubernetes's package manager](https://helm.sh/), does.
Helm takes files from a file system and converts those to Kubernetes resources using a mapping concept called [Charts](https://docs.helm.sh/developing_charts/#charts).

Helm Charts could potentially become be a replacement for Kubechain's adapters.

### Why not use Helm than?

#### Ease of use.
Kubechain is intended to be a suite. It allows you to get everything done in one go without having to use a set of tools individually.

#### Maintenance
Blockchain maintainers know it's code inside out and are better suited to develop and maintain a set of charts.
For instance, Hyperledger Burrow has it's own [set of charts](https://hub.kubeapps.com/charts/stable/burrow) developed and maintained by the team who created Burrow.

A note on the Burrow charts. I tested them between 2-4-2018 and 12-4-2018 and they: 
- Can't be generated on windows.
- Do't function on AWS.
- Don't function on GCE .
- For both AWS and GCE I mimicked the setup file from [this repo](https://github.com/kubernetes/charts/tree/master/stable/burrow/examples/1.human_deployment/README.md).

#### Code issues
Hyperledger Fabric has some issue which require alternate settings on individual nodes. Some of these have been resolved, such as [FAB-7428](https://jira.hyperledger.org/browse/FAB-7428), others are still being worked upon.
When I started to work on this project I had to work around these issues. Which I could only achieve by writing the code to do so myself.

#### Lack of control
Helm does not provide an API to interact with it. So I would have been forced to use shell commands, adding a dependency outside of Kubechain's management.
This would have resulted in a lack of control over Kubechain's dependencies. Which I'm not a fan of.

#### Effort. 
Aside from what's listed above, Helm only came to my attention when I was already knee-deep in this project.
Redirecting my efforts towards creating Helm charts instead of Adapters would have required me to familiarize myself with it's documentation and development tools.



