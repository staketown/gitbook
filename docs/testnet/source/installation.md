---
cover: ../../.gitbook/assets/source_banner.webp
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/source/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf source
git clone https://github.com/Source-Protocol-Cosmos/source.git
cd $HOME/source || return
git checkout v3.0.1
make install

sourced config keyring-backend os
sourced config chain-id sourcetest-1
sourced init "<Your moniker>" --chain-id sourcetest-1

curl -Ls https://snapshots-testnet.stake-town.com/source/genesis.json > $HOME/.source/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/source/addrbook.json > $HOME/.source/config/addrbook.json

APP_TOML="~/.source/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $APP_TOML

CONFIG_TOML="~/.source/config/config.toml"
SEEDS="51aa068c578243c470d846bed1b539369fdb0a14@rpc-t.source.nodestake.top:666"
PEERS="ace839c852739d1ea6e3675d30380fe085c1c23a@52.26.226.21:26656,8145d4d13511e7f89dbd257f51ed5d076941f12f@164.92.98.12:26656""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025usource"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install and configure cosmovisor...

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.source/cosmovisor/genesis/bin
mkdir -p ~/.source/cosmovisor/upgrades
cp ~/go/bin/sourced $HOME/.source/cosmovisor/genesis/bin

# Starting service and synchronization...

sudo tee /etc/systemd/system/sourced.service > /dev/null << EOF
[Unit]
Description=Source Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=sourced"
Environment="DAEMON_HOME=$HOME/.source"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

sourced tendermint unsafe-reset-all --home $HOME/.source --keep-addr-book

# Snapshots
URL="https://snapshots-testnet.stake-town.com/source/sourcetest-1_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.source
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.source/config/config.toml"
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
sudo systemctl enable sourced
sudo systemctl start sourced

sudo journalctl -u sourced -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.source/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.source/config/config.toml
sudo systemctl restart sourced && sudo journalctl -u sourced -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
sourced keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
sourced keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
sourced tx staking create-validator \
--amount=1000000usource \
--pubkey=$(sourced tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=sourcetest-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1usource \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
