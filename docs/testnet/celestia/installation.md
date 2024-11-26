---
cover: ../../.gitbook/assets/celestia-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/celestia/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd $HOME/celestia-app || return
git checkout v3.0.1-mocha

make install

celestia-appd config keyring-backend os
celestia-appd config chain-id mocha-4
celestia-appd init "Your Moniker" --chain-id mocha-4

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/celestia/genesis.json > $HOME/.celestia-app/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/celestia/addrbook.json > $HOME/.celestia-app/config/addrbook.json

APP_TOML="~/.celestia-app/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "nothing"|g' $APP_TOML
sed -i -e "s|^target_height_duration *=.*|timeout_commit = \"11s\"|" $HOME/.celestia-app/config/config.toml

CONFIG_TOML="~/.celestia-app/config/config.toml"
SEEDS=""
PEERS="006e6d1287b697a5d94fe7a832d7e8e72f9e838a@65.108.124.43:34656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.002utia"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.celestia-app/cosmovisor/genesis/bin
mkdir -p ~/.celestia-app/cosmovisor/upgrades
cp ~/go/bin/celestia-appd ~/.celestia-app/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/celestia-appd.service > /dev/null << EOF
[Unit]
Description=Celestia Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=celestia-appd"
Environment="DAEMON_HOME=$HOME/.celestia-app"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/celestia/mocha-4_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.celestia-app
[[ -f $HOME/.celestia-app/data/upgrade-info.json ]] && cp $HOME/.celestia-app/data/upgrade-info.json $HOME/.celestia-app/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.celestia-app/config/config.toml"
sed -i 's/timeout_propose =.*/timeout_propose = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote =.*/timeout_prevote = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit =.*/timeout_precommit = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_commit =.*/timeout_commit = "1s"/g' $CONFIG_TOML
sed -i 's/skip_timeout_commit =.*/skip_timeout_commit = false/g' $CONFIG_TOML
```

Enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd

sudo journalctl -u celestia-appd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.celestia-app/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.celestia-app/config/config.toml
sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
celestia-appd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
celestia-appd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
celestia-appd tx staking create-validator \
--amount=1000000utia \
--pubkey=$(celestia-appd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=mocha-4 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.002utia \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

# Install Bridge Node

Download and build binaries
```bash
cd $HOME
rm -rf celestia-node
git clone https://github.com/celestiaorg/celestia-node.git
cd celestia-node
git checkout v0.20.3-mocha
make build
sudo mv build/celestia $HOME/go/bin
make cel-key
sudo mv cel-key $HOME/go/bin
```

Add new bridge wallet
```bash
cel-key add bridge-wallet --node.type bridge --p2p.network mocha
```

Recover existent bridge wallet (in case you already have bridge wallet)
```bash
cel-key add bridge-wallet --node.type bridge --p2p.network mocha --recover
```

> ⚠️ Once you start the Bridge Node, a wallet key will be generated. You need to fund that address with testnet tokens (could be used faucet) to pay for PayForBlob transactions

Initialize Bridge node
```bash
celestia bridge init \
--keyring.keyname bridge-wallet \
--core.ip http://localhost \
--core.rpc.port 26657 \
--core.grpc.port 9090 \
--p2p.network mocha \
--rpc.port 26658 \
--gateway.port 26659
```

Create service
```bash
sudo tee /etc/systemd/system/celestia-bridge.service > /dev/null << EOF
[Unit]
Description=Celestia Bridge Node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which celestia) bridge start \\
--keyring.keyname bridge-wallet \\
--core.ip http://localhost \\
--core.rpc.port 26657 \\
--core.grpc.port 9090 \\
--p2p.network mocha \\
--rpc.port 26658 \\
--gateway.port 26659 \\
--metrics.tls=true \\
--metrics \\
--metrics.endpoint otel.celestia-mocha.com \\
--archival
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Load bridge node service file
```bash
sudo systemctl daemon-reload
sudo systemctl enable celestia-bridge.service
```

Start Bridge node
```bash
systemctl restart celestia-bridge.service
```

Check bridge node logs
```bash
journalctl -fu celestia-bridge.service -o cat
```

# Install Blobstream Orchestrator

Download and build binaries
```bash
cd $HOME
rm -rf orchestrator-relayer
git clone https://github.com/celestiaorg/orchestrator-relayer.git
cd orchestrator-relayer
git checkout v1.0.0
make build
sudo mv build/blobstream $HOME/go/bin
```

Initialize Orchestrator node
```bash
blobstream orchestrator init
```

Add Orchestrator EVM wallet
> The EVM private key needs to correspond to the EVM address provided when creating the validator.

ImportEVM wallet private key
```bash
blobstream orchestrator keys evm import ecdsa <YOUR PRIVATE KEY IN HEX FORMAT>
```

Register EVM address to your validator
```bash
celestia-appd tx qgb register \
<YOUR_VALIDATOR_ADDRESS> \
<EVM_ADDRESS> \
--from <YOUR_WALLET_ADDRESS> \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.002utia \
-y
```

Verify linked EVM address to your validator and save it variable
```bash
EVM_ADDRESS=celestia-appd query qgb evm $(celestia-appd keys show wallet --bech val -a)
echo $EVM_ADDRESS
```

Create service
```bash
sudo tee /etc/systemd/system/celestia-orchestrator.service > /dev/null << EOF
[Unit]
Description=Celestia Orchestrator service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which blobstream) orchestrator start \\
--core.grpc 127.0.0.1:9090 \\
--core.rpc tcp://127.0.0.1:26657 \\
--evm.account $EVM_ADDRESS \\
--evm.passphrase <YOUR_EVM_PASSPHRASE> \\
--p2p.bootstrappers /dns/bootstr-0-mocha-blobstream.celestia-mocha.com/tcp/30000/p2p/12D3KooWLrw6EQgDwvgqrqT8wLNJoQYN3SDAzaAxJgyiTa2xowyF
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Load orchestrator service file
```bash
sudo systemctl daemon-reload
sudo systemctl enable celestia-orchestrator
```

> Before running the orchestrator, make sure to have the indexing enabled on your RPC node. If the indexer was just activated, then, by default, it will not have the previous transactions indexed. And, if you run the orchestrator at the same time, it will try to create the commitments and will fail as the transactions are not indexed.

Start orchestrator node
```bash
systemctl start celestia-orchestrator
```

Check orchestrator node logs
```bash
journalctl -u celestia-orchestrator -f -o cat
```