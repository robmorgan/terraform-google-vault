# Vault Install Script

This folder contains a script for installing Vault and its dependencies. You can use this script, along with the
[run-vault script](https://github.com/hashicorp/terraform-google-vault/tree/master/modules/run-vault) it installs, to create a Vault [Google Image](
https://cloud.google.com/compute/docs/images) that can be deployed in [Google Cloud](https://cloud.google.com) across a
Managed Instance Group using the [vault-cluster module](https://github.com/hashicorp/terraform-google-vault/tree/master/modules/vault-cluster).

This script has been tested on the following operating systems:

* Ubuntu 16.04

There is a good chance it will work on other flavors of Debian as well.



## Quick start

To install Vault, use `git` to clone this repository at a specific tag (see the [releases page](../../../../releases) 
for all available tags) and run the `install-vault` script:

```
git clone --branch <VERSION> https://github.com/hashicorp/terraform-google-vault.git
terraform-google-vault/modules/install-vault/install-vault --version 0.5.4
```

The `install-vault` script will install Vault, its dependencies, and the [run-vault script](https://github.com/hashicorp/terraform-google-vault/tree/master/modules/run-vault).
You can then run the `run-vault` script when the server is booting to start Vault.

We recommend running the `install-vault` script as part of a [Packer](https://www.packer.io/) template to create a
Vault [Google Image](https://cloud.google.com/compute/docs/images) (see the [vault-consul-image example](
/examples/vault-consul-image) for sample code). You can then deploy the Image across a Managed Instance Group using the
[vault-cluster module](https://github.com/hashicorp/terraform-google-vault/tree/master/modules/vault-cluster) (see the [vault-cluster-public](https://github.com/hashicorp/terraform-google-vault/tree/master/examples/vault-cluster-public) and 
[vault-cluster-private](https://github.com/hashicorp/terraform-google-vault/tree/master/examples/vault-cluster-private) examples for fully-working sample code).




## Command line Arguments

The `install-vault` script accepts the following arguments:

* `version VERSION`: Install Vault version VERSION. Required. 
* `path DIR`: Install Vault into folder DIR. Optional.
* `user USER`: The install dirs will be owned by user USER. Optional.

Example:

```
install-vault --version 0.8.2
```



## How it works

The `install-vault` script does the following:

1. [Create a user and folders for Vault](#create-a-user-and-folders-for-vault)
1. [Install Vault binaries and scripts](#install-vault-binaries-and-scripts)
1. [Configure mlock](#configure-mlock)
1. [Install supervisord](#install-supervisord)
1. [Follow-up tasks](#follow-up-tasks)


### Create a user and folders for Vault

Create an OS user named `vault`. Create the following folders, all owned by user `vault`:

* `/opt/vault`: base directory for Vault data (configurable via the `--path` argument).
* `/opt/vault/bin`: directory for Vault binaries.
* `/opt/vault/data`: directory where the Vault agent can store state.
* `/opt/vault/config`: directory where the Vault agent looks up configuration.
* `/opt/vault/log`: directory where the Vault agent will store log files.
* `/opt/vault/tls`: directory where the Vault will look for TLS certs.


### Install Vault binaries and scripts

Install the following:

* `vault`: Download the Vault zip file from the [downloads page](https://www.vaultproject.io/downloads.html) (the 
  version number is configurable via the `--version` argument), and extract the `vault` binary into 
  `/opt/vault/bin`. Add a symlink to the `vault` binary in `/usr/local/bin`.
* `run-vault`: Copy the [run-vault script](https://github.com/hashicorp/terraform-google-vault/tree/master/modules/run-vault) into `/opt/vault/bin`. 


### Configure mlock

Give Vault permissions to make the `mlock` (memory lock) syscall. This syscall is used to prevent the OS from swapping
Vault's memory to disk. For more info, see: https://www.vaultproject.io/docs/configuration/#disable_mlock.


### Install supervisord

Install [supervisord](http://supervisord.org/). We use it as a cross-platform supervisor to ensure Vault is started
whenever the system boots and restarted if the Vault process crashes.


### Follow-up tasks

After the `install-vault` script finishes running, you may wish to do the following:

1. If you have custom Vault config (`.hcl`) files, you may want to copy them into the config directory (default:
   `/opt/vault/config`).
1. If `/usr/local/bin` isn't already part of `PATH`, you should add it so you can run the `vault` command without
   specifying the full path.
   


## Why use Git to install this code?

<!-- TODO: update the package managers URL to the final URL when this Blueprint is released -->

We needed an easy way to install these scripts that satisfied a number of requirements, including working on a variety 
of operating systems and supported versioning. Our current solution is to use `git`, but this may change in the future.
See [Package Managers](https://github.com/hashicorp/terraform-aws-consul/blob/master/_docs/package-managers.md) for
a full discussion of the requirements, trade-offs, and why we picked `git`.
