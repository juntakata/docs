# How to install and update bitcoind on Ubuntu

## Installing bitcoind
Type the following line to add the Bitcoin Personal Package Archive (PPA) to your system:

```
sudo apt-add-repository ppa:bitcoin/bitcoin
```

You will be prompted for your user password. Provide it to continue. Afterwards, the following text will be displayed:

```
Stable Channel of bitcoin-qt and bitcoind for Ubuntu, and their
dependencies

Note that you should prefer to use the official binaries, where possible, to
limit trust in Launchpad/the PPA owner.

No longer supports precise, due to its ancient gcc and Boost versions.
More info: https://launchpad.net/~bitcoin/+archive/ubuntu/bitcoin
```

Press [ENTER] to continue. Type the following line to get the most recent list of packages:

```
sudo apt-get update
```

A large number of lines will be displayed as different update files are downloaded. This step may take several minutes on a slow Internet connection.

To continue, choose one of the following options

1. To install the Bitcoin Core Graphical User Interface (GUI), type the following line and proceed to the Bitcoin Core GUI section below:

    ```
    sudo apt-get install bitcoin-qt
    ```

2. To install the Bitcoin Core daemon (bitcoind), which is useful for programmers and advanced users, type the following line and proceed to the Bitcoin Core Daemon section below:

    ```
    sudo apt-get install bitcoind
    ```

3. To install both the GUI and the daemon, type the following line and read both the GUI instructions and the daemon instructions. Note that you can’t run both the GUI and the daemon at the same time using the same configuration directory.

    ```
    sudo apt-get install bitcoin-qt bitcoind
    ```
 
 ## Updating bitcoind

```
bitcoin-cli stop
sudo apt-get update
sudo apt-get upgrade bitcoind
sudo reboot
```

## Starting bitcoind

```
bitcoind -daemon
```

## Stopping bitcoind

```
bitcoin-cli stop
```