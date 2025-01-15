---
cover: ../../.gitbook/assets/native-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/native/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/gonative
git clone https://github.com/gonative-cc/gonative.git
cd $HOME/gonative || return
git checkout v0.1.1

make build && mv $HOME/gonative/out/gonative $HOME/go/bin

gonative config set client keyring-backend os
gonative config set client chain-id native-t1
gonative init "Your Moniker" --chain-id native-t1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/native/genesis.json > $HOME/.gonative/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/native/addrbook.json > $HOME/.gonative/config/addrbook.json

APP_TOML="~/.gonative/config/app.toml"
# TODO: switch to pebble once it's fixed. https://github.com/cosmos/cosmos-sdk/issues/23133
sed -i "s/^app-db-backend *=.*/app-db-backend = \"goleveldb\"/" $APP_TOML
CONFIG_TOML="~/.gonative/config/config.toml"
sed -i "s/^db_backend *=.*/db_backend = \"goleveldb\"/" $CONFIG_TOML
SEEDS=""
PEERS="236946946eacbf6ab8a6f15c99dac1c80db6f8a5@65.108.203.61:52656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.1untiv"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.gonative/cosmovisor/genesis/bin
mkdir -p ~/.gonative/cosmovisor/upgrades
cp ~/go/bin/gonative ~/.gonative/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/gonatived.service > /dev/null << EOF
[Unit]
Description=Native Network Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=gonative"
Environment="DAEMON_HOME=$HOME/.gonative"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
gonative comet unsafe-reset-all --home $HOME/.gonative --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/native/native-t1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.gonative
[[ -f $HOME/.gonative/data/upgrade-info.json ]] && cp $HOME/.gonative/data/upgrade-info.json $HOME/.gonative/cosmovisor/genesis/upgrade-info.json
```

Enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable gonatived
sudo systemctl start gonatived

sudo journalctl -u gonatived -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.gonative/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.gonative/config/config.toml
sudo systemctl restart gonatived && sudo journalctl -u gonatived -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
gonative keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
gonative keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
gonative tx staking create-validator \
--amount=1000000untiv \
--pubkey=$(gonative comet show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=native-t1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=20000untiv \
-y
```