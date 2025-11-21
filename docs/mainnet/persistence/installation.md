---
cover: ../../.gitbook/assets/persistence-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/persistence/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/persistenceCore
git clone https://github.com/persistenceOne/persistenceCore.git
cd $HOME/persistenceCore || return
git checkout v16.0.0

make install

persistenceCore config keyring-backend os
persistenceCore config chain-id core-1
persistenceCore init "Your Moniker" --chain-id core-1

# Download genesis and addrbook
curl -Ls https://snapshots-1.stake-town.com/persistence/genesis.json > $HOME/.persistenceCore/config/genesis.json
curl -Ls https://snapshots-1.stake-town.com/persistence/addrbook.json > $HOME/.persistenceCore/config/addrbook.json

APP_TOML="~/.persistenceCore/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.persistenceCore/config/config.toml"
SEEDS=""
PEERS="aba2148170161c274d2d786bffbe6a692c535dfe@65.108.195.213:53656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.005uxprt"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.persistenceCore/cosmovisor/genesis/bin
mkdir -p ~/.persistenceCore/cosmovisor/upgrades
cp ~/go/bin/persistenceCore ~/.persistenceCore/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/persistenceCore.service > /dev/null << EOF
[Unit]
Description=Persistence Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=persistenceCore"
Environment="DAEMON_HOME=$HOME/.persistenceCore"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
persistenceCore tendermint unsafe-reset-all --home $HOME/.persistenceCore --keep-addr-book

URL=https://snapshots-1.stake-town.com/persistence/core-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.persistenceCore
[[ -f $HOME/.persistenceCore/data/upgrade-info.json ]] && cp $HOME/.persistenceCore/data/upgrade-info.json $HOME/.persistenceCore/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.persistenceCore/config/config.toml"
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
sudo systemctl enable persistenceCore
sudo systemctl start persistenceCore

sudo journalctl -u persistenceCore -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.persistenceCore/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.persistenceCore/config/config.toml
sudo systemctl restart persistenceCore && sudo journalctl -u persistenceCore -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
persistenceCore keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
persistenceCore keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
persistenceCore tx staking create-validator \
--amount=1000000uxprt \
--pubkey=$(persistenceCore tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=core-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.025uxprt \
--gas-adjustment=1.4 \
--gas=auto \
-y
```
