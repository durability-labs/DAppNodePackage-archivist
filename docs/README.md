# Dappnode package development

 1. [Description](#description)
 2. [Server](#server)
 3. [Install](#install)
 4. [Connect](#connect)
 5. [Considerations](#considerations)
 6. [Development](#development)
 7. [Publish](#publish)
 8. [Limitations](#limitations)
 9. [Known issues](#known-issues)


## Description

 Dappnode packages supports two types of [configuration](https://docs.dappnode.io/docs/dev/package-development/overview)
  - [Single Configuration](https://docs.dappnode.io/docs/dev/package-development/single-configuration) - Used to generate a single package, tailored for a specific configuration.
  - [Multi-Configuration](https://docs.dappnode.io/docs/dev/package-development/multi-configuration) - Used to generate multiple packages with varying configurations, such as different networks or client setups.

 Provided guide is focused on Multi-Configuration variant because it provides more flexibility.

 The easiest way to develop the package is to use a VM and in that guide we will use Hetzner Cloud.

 - [Docs](https://docs.dappnode.io)
 - [Package Development](https://docs.dappnode.io/docs/dev/package-development/overview)


## Server

 1. Run an Ubuntu VM on Hetzner - `8vCPU/16GB RAM` (`cx42/cpx41`)
 2. Create firewall rules based on the [Cloud Providers / AWS](https://docs.dappnode.io/docs/user/dappnode-cloud/providers/aws/set-up-instance/) guide

    | Protocol | Port         | Service       | Source      | Comment                              |
    | -------- | ------------ | ------------- | ----------- | ------------------------------------ |
    | `TCP`    | `22`         | `SSH`         | `0.0.0.0/0` |                                      |
    | `TCP`    | `80`         | `HTTP`        | `0.0.0.0/0` | Required for services exposing only? |
    | `TCP`    | `443`        | `HTTP`        | `0.0.0.0/0` | Required for services exposing only? |
    | `UDP`    | `51820`      | `Wireguard`   | `0.0.0.0/0` |                                      |
    | `TCP`    | `1024-65535` | `General TCP` | `0.0.0.0/0` |                                      |
    | `UDP`    | `1024-65535` | `General UDP` | `0.0.0.0/0` |                                      |


## Install

 1. We can install Dappnode on Ubuntu VM using [Script installation](https://docs.dappnode.io/docs/user/install/script/)
    ```shell
    # Prerequisites
    sudo wget -O - https://prerequisites.dappnode.io | sudo bash

    # Dappnode
    sudo wget -O - https://installer.dappnode.io | sudo bash

    # Dappnode profile
    source ~/.bashrc

    # Dappnode help
    dappnode_help
    ```


## Connect

 1. Check Dappnode status
    ```shell
    dappnode_status
    ```

 2. Get wireguard credentials and connect to the Dappnode instance - [WireGuard Access to Dappnode](https://docs.dappnode.io/docs/user/access-your-dappnode/vpn/wireguard/)
    ```shell
    dappnode_wireguard
    ```

 3. Open http://my.dappnode in the browser.


## Considerations

 1. Users might run a lot of different packages, which can use some standard ports like `30303`, this is why we used different default ports
    ```shell
    30303 --> 40303
    ```
    Just add 10000 to every port.


## Development

 1. Install [Node.js](Node.js) on Dappnode server using [nvm](https://github.com/nvm-sh/nvm)
    ```shell
    # nvm
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

    # Load
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm

    # Node 22
    nvm install 22

    # Check
    node --version
    # v22.20.0
    ```

 2. Install [DAppNodeSDK](https://github.com/dappnode/DAppNodeSDK) on remote Dappnode
    ```shell
    # Install
    npx -y @dappnode/dappnodesdk

    # Help
    npx @dappnode/dappnodesdk --help
    ```

 3. Clone GitHub repository on local machine
    ```shell
    git clone https://github.com/durability-labs/DAppNodePackage-archivist

    # For new package run 'init'
    # npx @dappnode/dappnodesdk init --use-variants --dir DAppNodePackage-archivist
    ```
    Add your changes to the code.

 4. Copy package files to Dappnode server
    ```shell
    local_dir="DAppNodePackage-archivist"
    remote_dir="/opt/DAppNodePackage-archivist"
    host="root@<server-ip>"

    rsync -avze ssh --rsync-path='sudo rsync --mkpath' "${local_dir}/" "${host}:${remote_dir}/" --delete
    ```

 5. Get Dappnode IPFS IP via UI or CLI
    - `Packages` --> `System packages` --> `Ipfs` --> `Network` --> `Container IP`
    - `docker inspect --format '{{ .NetworkSettings.Networks.dncore_network.IPAddress }}' DAppNodeCore-ipfs.dnp.dappnode.eth`

 6. Build the package
    ```shell
    # Code directory - multi-arch builds failed with --dir argument
    cd /opt/DAppNodePackage-archivist

    # Use Ipfs node IP
    npx @dappnode/dappnodesdk build --variants archivist --provider=http://172.33.0.3:5001
    # npx @dappnode/dappnodesdk build --all-variants --provider=http://172.33.0.3:5001
    ```
    ```
    Dappnode Package (archivist.public.dappnode.eth) built and uploaded
    DNP name : archivist.public.dappnode.eth
    Release hash : /ipfs/QmSpQZbHTiPTeg3mEkWWoERhMjhiufjhhfegS2ExkVkMWw
    http://my.dappnode/installer/public/%2Fipfs%2FQmSpQZbHTiPTeg3mEkWWoERhMjhiufjhhfegS2ExkVkMWw
    ```

 7. Install the package via DAppStore and using IPFS CID from previous step and check `Bypass only signed safe restriction`


## Publish

 - [Package Publishing](https://docs.dappnode.io/docs/dev/package-publishing/publish-packages-clients)


## Limitations

 1. Dappnode packages are built on top of the [Docker Compose](https://docs.docker.com/compose/) which has limited configuration flexibility and DAppNodeSDK does not provide any useful workarounds.

 2. Docker Compose base imply the following limitations
    - Variables
      - If we need to pass an optional environment variable, it needs to be defined in Compose file with some default value and it anyway will be passed to the container
      - If that optional variable can't accept a blank value, we should undefine it conditionally in the Docker entrypoint
    - Ports
      - If we need to define an optional port forwarding, it needs to be defined in Compose file with some default values and it anyway will be active and take the port on the node
      - There is no way to configure "really optional" port forwarding
      - A workaround would be use a separate package variant, but it is to big overhead

 3. We can't have a relation between variable and port forwarding, to setup same value using a single field. User have to fill separately two fields with the same value.

 4. Multi-Configuration package does not provide a real flexibility, it just generate multiple separate packages and it doesn't work like a single package with multiple options during the installation.

 5. [Using profiles with Compose](https://docs.docker.com/compose/how-tos/profiles/) is not supported.

 6. There is no way to setup a custom service name during package installation and it is predefined in the main `dappnode_package.json`
    - We can set an alias like `archivist.public.dappnode --> archivist-app.archivist.public.dappnode`
    - That can be done for a single service in the package

 7. Is there a way to adjust container port for [Published ports](https://docs.docker.com/engine/network/#published-ports) or we can configure just host port?

 8. File [`setup-wizard.yml`](<https://docs.dappnode.io/docs/dev/references/setup-wizard>) is not supported in [Multi-Config Package Development](<https://docs.dappnode.io/docs/dev/package-development/multi-configuration>) which is very confusing. And same issue is with the `getting-started.md`.

 9. There is no way to setup custom service names for multiple services and they all namespaced under the `package name`
    ```shell
    # Public packages
              geth.archivist.public.dappnode
     archivist-app.archivist.public.dappnode
    archivist-node.archivist.public.dappnode
    ```

 1. When we have Multi-Configuration package, we should define different package name for each variant, which imply different namespaces for services names and that looks not so nice, for example
    ```shell
    # Package archivist
                   archivist.public.dappnode --> archivist-app.archivist.public.dappnode
     archivist-app.archivist.public.dappnode
    archivist-node.archivist.public.dappnode

    # Package archivist-local-geth
                   archivist-local-geth.public.dappnode --> archivist-app.archivist-local-geth.public.dappnode
     archivist-app.archivist-local-geth.public.dappnode
    archivist-node.archivist-local-geth.public.dappnode
              geth.archivist-local-geth.public.dappnode
    ```
    If we would like to have separate packages, which would permit to use same handy URL like `archivist.public.dappnode` for main service, and other services under that namespace, it would be required to have separate repositories(package folders) with the same package name. It can be a cosmetic point, but it highlights a limitation we have.


## Known issues

 1. During local package build it is uploaded to the local IPFS node, but in the Dappnode UI package avatar is loaded from the https://gateway.ipfs.dappnode.io, so most probably it will not be shown and it is not so clear what is the issue. Maybe something is wrong with avatar or something else? We can use default avatar, which is known by Dappnode IPFS gateway.

 2. File `getting-started.md` is not specified in the official documentation, but it exists and is very usefully.

 3. Dappnodesdk does not support `compose.yaml` file, [which is default and preferred](https://docs.docker.com/compose/intro/compose-application-model/).

 4. Often time it can be more effective to [explorer existing packages](https://github.com/dappnode?q=DAppNodePackage&type=all&language=&sort=) configuration, than to use a documentation.

 5. During the package build, Docker warn that ["the attribute `version` is obsolete"](https://docs.docker.com/reference/compose-file/version-and-name/#version-top-level-element-obsolete), but dappnodesdk will fail if we remove it - that is very confusing.
