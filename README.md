# Atlas Mist Browser

[![Join the chat at https://gitter.im/AtlasBlockchain/AtlasMist](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/atlas/mist?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status develop branch](https://travis-ci.org/atlas/mist.svg?branch=develop)](https://travis-ci.org/atlas/mist)
[![Code Climate](https://codeclimate.com/github/ethereum/mist/badges/gpa.svg)](https://codeclimate.com/github/atlas/mist)

The Atlas Mist browser is the tool of choice to browse and use Ðapps on the Atlas Blockchain.

For the Atlas Mist API see the [MISTAPI.md](MISTAPI.md).

Please note that this repository is the Electron host for the Meteor based wallet dapp whose repository is located here: https://github.com/ethereum/meteor-dapp-wallet.

## Help and troubleshooting

Please check the [Atlas Mist troubleshooting guide](https://github.com/atlaswork/mist/wiki).

Or the [Gitter Channel](https://gitter.im/AtlasBlockchain/AtlasMist), to connect with the community for instant help.

## Installation

If you want to install the app from a pre-built version on the [release page](https://github.com/atlaswork/atlasmist/releases),
you can simply run the executeable after download.

For updating simply download the new version and copy it over the old one (keep a backup of the old one if you want to be sure).

#### Config folder
The data folder for Mist is stored in other places:

- Windows `%APPDATA%\AtlasMist`
- macOS `~/Library/Application\ Support/AtlasMist`
- Linux `~/.config/AtlasMist`


## Development

For development, a Meteor server will need to be started to assist with live reload and CSS injection.
Once a Mist version is released the Meteor frontend part is bundled using the `meteor-build-client` npm package to create pure static files.

### Dependencies

To run mist in development you need:

- [Node.js](https://nodejs.org) `v7.x` (use the prefered installation method for your OS)
- [Meteor](https://www.meteor.com/install) javascript app framework
- [Yarn](https://yarnpkg.com/) package manager
- [Electron](http://electron.atom.io/) `v1.4.15` cross platform desktop app framework
- [Gulp](http://gulpjs.com/) build and automation system

Install the latter ones via:

    $ curl https://install.meteor.com/ | sh
    $ curl -o- -L https://yarnpkg.com/install.sh | bash
    $ yarn global add electron@1.4.15
    $ yarn global add gulp

### Initialisation

Now you're ready to initialise Mist for development:

    $ git clone https://github.com/atlaswork/atlasmist.git
    $ cd AtlasMist
    $ yarn

To update Mist in the future, run:

    $ cd AtlasMist
    $ git pull
    $ yarn

### Run Mist

For development we start the interface with a Meteor server for autoreload etc.
*Start the interface in a separate terminal window:*

    $ cd AtlasMist/interface && meteor --no-release-check

In the original window you can then start Mist with:

    $ cd AtlasMist
    $ yarn dev:electron

*NOTE: client-binaries (e.g. [geth](https://github.com/atlaswork/go-atlas)) specified in [clientBinaries.json](https://github.com/atlas/mist/blob/master/clientBinaries.json) will be checked during every startup and downloaded if out-of-date, binaries are stored in the [config folder](#config-folder)*

*NOTE: use `--help` to display available options, e.g. `--loglevel debug` (or `trace`) for verbose output*

### Run the Wallet

Start the wallet app for development, *in a separate terminal window:*

    $ cd AtlasMist/interface && meteor --no-release-check

    // and in another terminal

    $ cd my/path/meteor-dapp-wallet/app && meteor --port 3050

In the original window you can then start Mist using wallet mode:

    $ cd AtlasMist
    $ yarn dev:electron --mode wallet


### Connecting to node via HTTP instead of IPC

This is useful if you have a node running on another machine, though note that
it's less secure than using the default IPC method.

```bash
$ yarn dev:electron --rpc http://localhost:57200
```


### Passing options to Geth

You can pass command-line options directly to Geth by prefixing them with `--node-` in
the command-line invocation:

```bash
$ yarn dev:electron --mode atlasmist --node-rpcport 8080 --node-networkid 9082075
```

The `--rpc` AtlasMist option is a special case. If you set this to an IPC socket file
path then the `--ipcpath` option automatically gets set, i.e.:

```bash
$ yarn dev:electron --rpc /my/geth.ipc
```

...is the same as doing...


```bash
$ yarn dev:electron --rpc /my/geth.ipc --node-ipcpath /my/geth.ipc
```

### Creating a local private Atlas blockchain for developing dApps

See this guide to quickly set up a local private Atlas Blockchain network on your computer:
....coming soon


### Using Atlas Mist with a privatenet

To run a private network you will need to set the IPC path, network id and data
folder:

```bash
$ yarn dev:electron --rpc ~/Library/Atlas/geth.ipc --node-networkid 9082075 --node-datadir ~/Library/Atlas/privatenet
```

_NOTE: since `ipcpath` is also a Atlas Mist option you do not need to also include a
`--node-ipcpath` option._

You can also launch `geth` separately with the same options prior starting
Mist.


### Deployment

Our build system relies on [gulp](http://gulpjs.com/) and [electron-builder](https://github.com/electron-userland/electron-builder/).

#### Dependencies

[meteor-build-client](https://github.com/frozeman/meteor-build-client) bundles the [meteor](https://www.meteor.com/)-based interface. Install it via:

    $ npm install -g meteor-build-client

Furthermore cross-platform builds require additional [`electron-builder` dependencies](https://github.com/electron-userland/electron-builder/wiki/Multi-Platform-Build#linux). On macOS those are:

    // windows deps
    $ brew install wine --without-x11 mono makensis

    // linux deps
    $ brew install gnu-tar libicns graphicsmagick xz

#### Generate packages

To generate the binaries for Mist run:

    $ gulp

To generate the Atlas Wallet (this will pack the one Ðapp from https://github.com/atlaswork/meteor-dapp-wallet):

    $ gulp --wallet

The generated binaries will be under `dist_mist/release` or `dist_wallet/release`.


#### Options

##### platform

To build binaries for specific platforms (default: all available) use the following flags:

    // on mac
    $ gulp --win --linux --mac

    // on linux
    $ gulp --win --linux

    // on win
    $ gulp --win

##### walletSource

With the `walletSource` you can specify the Wallet branch to use, default is `master`:

    $ gulp --wallet --walletSource develop


Options are:

- `master`
- `develop`
- `local` Will try to build the wallet from [mist/]../meteor-dapp-wallet/app

*Note: applicable only when combined with `--wallet`*

#### skipTasks

When building a binary, you can optionally skip some tasks — generally for testing purposes.

  $ gulp --mac --skipTasks=bundling-interface,release-dist

#### Checksums

Spits out the MD5 checksums of distributables.

It expects installer/zip files to be in the generated folders e.g. `dist_mist/release`

    $ gulp checksums [--wallet]


## Testing

Tests are ran using [Spectron](https://github.com/electron/spectron/), a webdriver.io runner built for Electron.

First make sure to build Mist with:

    $ gulp

Then run the tests:

    $ gulp test

*Note: Integration tests are not yet supported on Windows.*
