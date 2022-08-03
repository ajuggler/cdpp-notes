---
label: Setting up the Cardano Node
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -1
---


<a id="org605f966"></a>

# Installing the Cardano components on **MacOS**

These are the steps that worked for me.  My machine is an **Intel**
based Mac.  For reference I also include the extra steps needed for an
**M1** based Mac.

(Based on [Installing cardano-node
instructions](https://developers.cardano.org/docs/get-started/installing-cardano-node/)
at [developers.cardano.org](https://developers.cardano.org/), with
some modifications.)


<a id="org3a1ef57"></a>

## XCode

[Downloaded](https://developer.apple.com/xcode/) Xcode and the Xcode
Command Line Tools.  Since my computer runs MacOS Catalina, I downloaded
version 12.4.  [Homebrew](https://brew.sh/) needs to also be installed.


<a id="orge18f16d"></a>

## Installing Homebrew packages

Installed some libraries with `brew`:

    brew install jq
    brew install libtool
    brew install autoconf
    brew install automake
    brew install pkg-config

!!!
If you have an M1 Mac, you will need to also install llvm.

    brew install llvm
!!!

!!!
You need to have GHC 8.10.7 and Cabal 3.6.2.0 installed on your computer.  You can verify that you have the correct versions with

    ghc --version
    cabal --version

If you need to upgrade GHC (or install from scratch), please refer to the corresponding instructions [here](https://developers.cardano.org/docs/get-started/installing-cardano-node#macos) .
!!!

<a id="orgc39a31c"></a>

## Downloading & Compiling


<a id="orgda1db8c"></a>

### Working directory

Created a working directory to store the source-code and build the
components.

    mkdir -p $HOME/cardano-src
    cd $HOME/cardano-src


<a id="org0e00027"></a>

### Install **libsodium**

Next, I downloaded, compiled and installed `libsodium`:

    git clone https://github.com/input-output-hk/libsodium
    cd libsodium
    git checkout 66f017f1
    ./autogen.sh
    ./configure
    make
    sudo make install


<a id="org8458164"></a>

### Added to bash shell profile

Added a couple of lines to my bash shell profile `$HOME/.bashrc`

    export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
    export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"

Reloaded the shell profile with `source $Home/.bashrc`.  (If you use *zsh*, use `.zshrc` whenever I refer to `.bashrc`.)

!!!
If your system is running on an M1 Mac (so that you also installed `llvm`), then you will also need to add

    export PATH="/opt/homebrew/opt/llvm/bin:$PATH"

and reload the shell profile with `source $HOME/.bashrc` .
!!!


<a id="org21ecaed"></a>

### Install `libsecp256k1`

    cd $HOME/cardano-src
    git clone https://github.com/bitcoin-core/secp256k1
    cd secp256k1
    git checkout ac83be33
    ./autogen.sh
    ./configure --enable-module-schnorrsig --enable-experimental
    make
    make check
    sudo make install


<a id="orgc1ba8f6"></a>

## Install the Cardano components

-   Next, I downloaded, compiled and installed `cardano-node` and `cardano-cli`:

        cd $HOME/cardano-src

-   If there exists a directory named `cardano-node`, say from a previous version, remove it (with all its contents).

-   Downloaded the `cardano-node` respository:

        git clone https://github.com/input-output-hk/cardano-node.git
        cd cardano-node
        git fetch --all --recurse-submodules --tags

Ordinarily, we switch the repository to the latest tagged commit with:

    git checkout $(curl -s https://api.github.com/repos/input-output-hk/cardano-node/releases/latest | jq -r .tag_name)

However, these are not ordinary times, as we are in the middle of a protocol upgrade (as of 2022-08-01).  In order to be able to work with the testnet, we instead force a switch to either version 1.35.1 or 1.35.2:

    git checkout -f tags/1.35.2


<a id="org2a960f7"></a>

## Build the Cardano components

-   We explicitly force `ghc` to use the correct version:

        cabal configure --with-compiler=ghc-8.10.7

!!!
If your system is running on an M1 Mac then you will also need to set these cabal related options before building:

    echo "package trace-dispatcher" >> cabal.project.local
    echo "  ghc-options: -Wwarn" >> cabal.project.local
    echo "" >> cabal.project.local
!!!

-   Built the node

        cabal build all

-   Install the newly built node and CLI to `$HOME/.local/bin`

        mkdir -p $HOME/.local/bin
        cp -p "$(./scripts/bin-path.sh cardano-node)" $HOME/.local/bin/
        cp -p "$(./scripts/bin-path.sh cardano-cli)" $HOME/.local/bin/

-   Added the following line to the shell profile `$HOME/.bashrc`

        export PATH="$HOME/.local/bin/:$PATH"

and reloaded with  `source $HOME/.bashrc` 

-   To verfiy all is OK, checked versions installed:

        cardano-cli --version
        cardano-node --version


<a id="org9be11d3"></a>

# Running cardano-node (with external hard drive)

You need to create a directory for the configuration files.  In my case, I created

    mkdir $HOME/code/haskell/emurgo/testnet
    cd $HOME/code/haskell/emurgo/testnet


<a id="orgbebff7c"></a>

## Configuration files

From this [repository](https://hydra.iohk.io/build/13695229/download/1/index.html), download the following configuration files:

-   `testnet-config.json`
-   `testnet-byron-genesis.json`
-   `testnet-shelley-genesis.json`
-   `testnet-alonzo-genesis.json`
-   `testnet-topology.json`


<a id="org4cadb9d"></a>

## Socket

Define the following environment variable:

    export CARDANO_NODE_SOCKET_PATH=$HOME/code/haskell/emurgo/testnet/node.socket

(The Cardano socket is created automatically when you start the Cardano node for the first time.)

!!!
Make sure to write '`export CARDANO_NODE_SOCKET_PATH`' **without** a dollar sign in front of `CARDANO`.
!!!

!!!
You need to export this environment variable whenever you start a shell in which you intend to run the script `start0node-testnet.sh` defined in the next subsection.
!!!


<a id="org5602a30"></a>

## Script for starting the `cardano-node`

-   Create the script `start-node-testnet.sh` with the following content:

    ```
    cardano-node run \
     --topology testnet-topology.json \
     --database-path db \
     --socket-path node.socket \
     --host-addr 127.0.0.1 \
     --port 3001 \
     --config testnet-config.json
     ```

-   Make the script executable with

        chmod u+x start-node-testnet.sh


<a id="org29e5ed7"></a>

## Database on an external hard drive

The `database-path` sets the name of the database directory.  For me it
was desirable to have the database on an **external hard drive**.  This is
achieved simply by setting a symbolic link.

Assume we want the database to sit inside directory `cardano` at the
root of an external hard drive named *External-Hard-Drive*.  Remembering
that the active directory on our shell is still `testnet`, we execute

    mkdir -p /Volume/External-Hard-Drive/cardano/db
    ln -s /Volume/External-Hard-Drive/cardano/db .


<a id="orgbcd2838"></a>

## Start the Cardano node

After the symbolic link was created, I started the Cardano node with

    ./start-node-testnet.sh

My MacBook took several hours to sync the testnet.


<a id="org71b8275"></a>

## Verify sync status

Verified the status of synchronization with

    cardano-cli query tip --testnet-magic 1097911063

which, after several hours, resulted in

    {
        "block": 3749636,
        "epoch": 220,
        "era": "Babbage",
        "hash": "7b0d1965263626137c4f5cd70b3fb238765ddd499b45fea425e4d6aa735d589a",
        "slot": 64992566,
        "syncProgress": "100.00"
    }


<a id="org3dd77c6"></a>

# Reference

The repository of the cardano node components sits at 
<https://github.com/input-output-hk/cardano-node> .

