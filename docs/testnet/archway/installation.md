---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/archway/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/archway
git clone https://github.com/archway-network/archway.git
cd $HOME/archway || return
git checkout v9.0.0-rc4

make install

archwayd config keyring-backend os
archwayd config chain-id constantine-3
archwayd init "Your Moniker" --chain-id constantine-3

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/archway/genesis.json > $HOME/.archway/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/archway/addrbook.json > $HOME/.archway/config/addrbook.json

APP_TOML="~/.archway/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.archway/config/config.toml"
SEEDS=""
PEERS="d1334258b592ebccb85a917aa65976b74e254a60@65.109.65.248:31656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "1000000000000aconst"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.archway/cosmovisor/genesis/bin
mkdir -p ~/.archway/cosmovisor/upgrades
cp ~/go/bin/archwayd ~/.archway/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/archwayd.service > /dev/null << EOF
[Unit]
Description=Archway Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=archwayd"
Environment="DAEMON_HOME=$HOME/.archway"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
archwayd tendermint unsafe-reset-all --home $HOME/.archway --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/archway/constantine-3_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.archway
[[ -f $HOME/.archway/data/upgrade-info.json ]] && cp $HOME/.archway/data/upgrade-info.json $HOME/.archway/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.archway/config/config.toml"
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
sudo systemctl enable archwayd
sudo systemctl start archwayd

sudo journalctl -u archwayd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.archway/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.archway/config/config.toml
sudo systemctl restart archwayd && sudo journalctl -u archwayd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
archwayd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
archwayd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
archwayd tx staking create-validator \
--amount=1000000000000000000aconst \
--pubkey=$(archwayd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=constantine-3 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=1000000000000aconst \
--gas-adjustment=1.4 \
--gas=auto \
-y
```
