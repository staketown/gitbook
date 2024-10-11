---
cover: ../../.gitbook/assets/arkeo-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/arkeo/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/arkeo
git clone https://github.com/arkeonetwork/arkeo.git
cd arkeo || return
git checkout master

TAG=testnet make install 

arkeod config keyring-backend os
arkeod config chain-id arkeo
arkeod init "Your Moniker" --chain-id arkeo-testnet-3

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/arkeo/genesis.json > $HOME/.arkeo/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/arkeo/addrbook.json > $HOME/.arkeo/config/addrbook.json

APP_TOML="~/.arkeo/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.arkeo/config/config.toml"
SEEDS="9dfa5f2d19c1174baf5e597965394269e654f9b7@seed31.innovationtheory.com:26656"
PEERS="e6b058d1d6be000d67b87e9d11cb0de1bba1e477@65.109.65.248:42656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uarkeo"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.arkeo/cosmovisor/genesis/bin
mkdir -p ~/.arkeo/cosmovisor/upgrades
cp ~/go/bin/arkeod ~/.arkeo/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.arkeo
ExecStart=$(which cosmovisor) run start --home $HOME/.arkeo
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=arkeod"
Environment="DAEMON_HOME=$HOME/.arkeo"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
arkeod tendermint unsafe-reset-all --home $HOME/.arkeo --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/arkeo/arkeo-testnet-3_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.arkeo
[[ -f $HOME/.arkeo/data/upgrade-info.json ]] && cp $HOME/.arkeo/data/upgrade-info.json $HOME/.arkeo/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.arkeo/config/config.toml"
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
sudo systemctl enable arkeod
sudo systemctl start arkeod

sudo journalctl -u arkeod -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.arkeo/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.arkeo/config/config.toml
sudo systemctl restart arkeod && sudo journalctl -u arkeod -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
arkeod keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
arkeod keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
arkeod tx staking create-validator \
--amount=1000000uarkeo \
--pubkey=$(arkeod tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=arkeo-testnet-3 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=500uarkeo \
-y
```
