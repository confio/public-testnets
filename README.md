If you find this Repo, be aware it is a semi-public testnet. \
We have already assigned external validators for the task and not accepting newcomers. Thank you

# Tgrade - Dryrunnet

You can see the live network via our [block explorer](https://dryrunnet.aneka.io). (Soon) \
When you are ready to build a node, follow the instructions below:

## Hardware Requirements
For running a tgrade validator. We tested successfully with the following Architecture:

- Ubuntu 20.04 LTS
- go version 1.18 [1] or newer
- Installed packages make and build-essential [2] [3]
- 2 or more CPU cores Intel or AMD chipset
- At least 40GB of disk storage
- At least 4GB of memory (RAM)

Ref - \
[1] https://github.com/golang/go/wiki/Ubuntu \
[2] https://packages.ubuntu.com/focal/make \
[3] https://packages.ubuntu.com/focal/build-essential

You can use a physical infrastructure (baremetal) or wellknown cloud providers like: DigitalOcean, AWS, Google Cloud Platform, among others

## Build the tgrade binary
The tgrade binary is the backbone of the platform. It is both blockchain node and interaction client. Therefore we use git to clone tgrade repo into our running validator
```bash
git clone https://github.com/confio/tgrade
cd tgrade
git checkout v0.9.0
```

Run GO install and build for the upcoming binary
```bash
make build
```

Move the binary to an executable path
```bash
sudo mv build/tgrade /usr/local/bin
```

## Setting up a Genesis Tgrade Validator - PHASE 1

### Initialize your genesis and configuration files
Initialize your genesis and configuration files for all validators nodes

Usage:
```bash
tgrade init [moniker] [flags]
tgrade init my-validator --chain-id tgrade-dryrunnet --home /opt/validator/.tgrade
```

### Import your Validator Key
We already have assigned a few external validators from this task. Therefore use the tgrade address YOU provided via Email.

Usage:
```bash
tgrade keys add <name> --recover
tgrade keys add my-validator --recover --home /opt/validator/.tgrade
```

Into the mnemonic(s) used for your tgrade address

### Get the pre-genesis file
Get the genesis file and moved to the right location
```bash
wget https://raw.githubusercontent.com/confio/public-testnets/main/dryrunnet/config/pre-genesis.json -O ~/opt/validator/.tgrade/config/genesis.json
```
( this will be the case if the APP Home directory is /opt/validator/.tgrade , please change it accordingly to your system/validator)

### Setup the right parameters and values on the TOML files
Please edit the `config/app.toml` and `config/config.toml` accordingly

```
- app.toml: set minimum-gas-prices
  minimum-gas-prices = "0.05utgd"

- config.toml: set persistent_peers and other suggested changes
  moniker = "<your validator name>"
  persistent_peers = "9c70e7fb4237de7dfb842c51d0c8a2bee6b843c0@168.119.252.165:26656,f0976ec13d3498397b0a891b44c9a024f8eebb4a@188.34.162.243:26656,ee664babe18b1005fee0548c8818143e745ad80a@142.132.225.3:26656"
```

## Create genesis txs - PHASE 2

### Collect tendermint node-ids and validator info
We need to collect from the genesis validators:
* node-id    ```tgrade tendermint show-node-id```
* pubkey     ```tgrade tendermint show-validator```
* IP and port to be used

### Create genesis txs
On each validator we need to create a genesis tx, by running:
```bash
tgrade gentx my-validator 0utgd 90000000utgd \
  --amount 0utgd \
  --vesting-amount 90000000utgd \
  --fees 10000utgd \
  --moniker my-validator \
  --ip "<public ip>" \
  --node-id $(tgrade tendermint show-node-id) \
  --chain-id tgrade-dryrunnet \
  --home /opt/validator/.tgrade
```
node-id, pubkey and home values are just examples, please change it accordingly to your system/validator

### Upload your Gen_TX
The above will create a gentx file. We are going to need it for the genesis collect.
1. Fork the repo: https://github.com/confio/public-testnets , clicking on fork, and choose your account
2. Clone your fork copy to your local machine
3. Copy the gentx file into `../dryrunnet/config/gentx/`
4. Commit and push the repo
5. Create a pull request from your fork to the main repo
6. Inform us on the discord channel

```bash
git clone https://github.com/<your_github_username>/public-testnets
cd public-testnets
git add public-testnets/dryrunnet/config/gentx/
git commit -am "<your validator name> - gentx - comment"
git push origin master
```

## Start your Validator - PHASE 3

### Get the final genesis file
Once we gather all the genesis txs. We will create the final version of the genesis file to be used on the network

Get the genesis file and moved to the right location
```bash
wget https://raw.githubusercontent.com/confio/public-testnets/main/dryrunnet/config/genesis.json -O ~/.tgrade/config/genesis.json
```
( this will be the case if the APP Home directory is ~/.tgrade , please change it accordingly to your system/validator)

### Start the syncing
There are different ways to manage the tgrade binary on your validator,
1. Setting up such binary to be managed by systemd, or
2. Open a tmux or screen session and run tgrade start

The syntax is:
```bash
tgrade start --rpc.laddr tcp://0.0.0.0:26657 --home /opt/validator/.tgrade
```
( this will be the case if the APP Home directory is /opt/validator/.tgrade , please change it accordingly to your system/validator)

