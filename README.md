# Archivist Dappnode packages

 <p align="center" width="100%">
   <img src="avatar-archivist.png" alt="Archivist Dappnode package" />
 </p>


## Description

 [Archivist](https://archivist.storage) is a durable, decentralised storage protocol designed to safeguard the world's most valuable information.

 Repository contains code to build Dappnode packages for Archivist
 - `Archivist`

 We should select `Archivist` package in case we would like to run Archivist and use a Public RPC endpoint.


## Installation

 > [!NOTE]
 > This code is in a **preview mode** and can be installed only from local builds. Please check [Development](docs/README.md#development) documentation for more details.
 ```shell
 # Clone
 git clone https://github.com/durability-labs/DAppNodePackage-archivist

 # Directory
 cd DAppNodePackage-archivist

 # Build specific package (~ 10 minutes) - use your IPFS package Container IP
 npx @dappnode/dappnodesdk build --variants archivist --provider=http://172.33.0.3:5001
 ```

 | Package   | Link                                                                                                                                                   |
 | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
 | Archivist | [`/ipfs/QmVSTCz63GDG3M2FvdCimNmrrNLrrtG16X1m86mxgdDaQR`](http://my.dappnode/installer/public/%2Fipfs%2FQmVSTCz63GDG3M2FvdCimNmrrNLrrtG16X1m86mxgdDaQR) |


## Todo

 1. Automate package publishing using [GitHub Actions](https://docs.dappnode.io/docs/dev/github-actions).
 2. Add metrics support based on the [DAppNodePackage-DMS](https://github.com/dappnode/DAppNodePackage-DMS).


## Known issues

 1. Dappnode [Setup Wizard](https://docs.dappnode.io/docs/dev/references/setup-wizard) and UI does not permit to update port mapping port for container. And in that case we are using a default value, which is set to `8090` and appropriate Archivist variable `ARCHIVIST_DISC_PORT=8090` - these values are **hardcoded** in the [docker-compose.yml](docker-compose.yml).

    This also is a Archivist limitation, because for DHT peers connection, we are using `src-ip:src-port` information and this can't be changed now. So, listener port must be equal with the forwarded one.

    A single way to bypass that is to update, after installation, `ARCHIVIST_DISC_PORT` variable to the new value and add a new port forwarding in the the package Network settings.

 2. Dappnode [Setup Wizard](https://docs.dappnode.io/docs/dev/references/setup-wizard) is not supported for [Multi-Config Package](https://docs.dappnode.io/docs/dev/package-development/multi-configuration), so our main *setup-wizard.yml* does not contain any settings for Geth configuration and we are using all hardcoded values.
