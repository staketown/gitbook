---
cover: ../../.gitbook/assets/prysm-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/prysm/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/prysm
git clone https://github.com/kleomedes/prysm.git
cd $HOME/prysm || return
git checkout v0.1.0-devnet

make install

prysmd config keyring-backend os
prysmd config chain-id prysm-devnet-1
prysmd init "Your Moniker" --chain-id prysm-devnet-1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/prysm/genesis.json > $HOME/.prysm/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/prysm/addrbook.json > $HOME/.prysm/config/addrbook.json

APP_TOML="~/.prysm/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.prysm/config/config.toml"
SEEDS=""
PEERS="69509925a520c5c7c5f505ec4cedab95073388e5@136.243.13.36:29856,bc1a37c7656e6f869a01bb8dabaf9ca58fe61b0c@5.9.73.170:29856,b377fd0b14816eef8e12644340845c127d1e7d93@79.13.87.34:26656,c80143f844fd8da4f76a0a43de86936f72372168@184.107.57.137:18656,afc7a20c15bde738e68781238307f4481938109d@94.130.35.120:18656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0uprysm"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.prysm/cosmovisor/genesis/bin
mkdir -p ~/.prysm/cosmovisor/upgrades
cp ~/go/bin/prysmd ~/.prysm/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/prysmd.service > /dev/null << EOF
[Unit]
Description=Prysm Network Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=prysmd"
Environment="DAEMON_HOME=$HOME/.prysm"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
prysmd tendermint unsafe-reset-all --home $HOME/.prysm --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/prysm/prysm-devnet-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.prysm
[[ -f $HOME/.prysm/data/upgrade-info.json ]] && cp $HOME/.prysm/data/upgrade-info.json $HOME/.prysm/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.prysm/config/config.toml"
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
sudo systemctl enable prysmd
sudo systemctl start prysmd

sudo journalctl -u prysmd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.prysm/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.prysm/config/config.toml
sudo systemctl restart prysmd && sudo journalctl -u prysmd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
prysmd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
prysmd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
prysmd tx staking create-validator \
--amount=1000000uprysm \
--pubkey=$(prysmd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=prysm-devnet-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.0002uprysm \
--gas-adjustment=1.6 \
--gas=auto \
-y
```
