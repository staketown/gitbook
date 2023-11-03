---
cover: ../../.gitbook/assets/banner_cleanup.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/lava/install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf lava
git clone https://github.com/lavanet/lava
cd $HOME/lava || return
git checkout v0.26.1
make install

lavad config keyring-backend os
lavad config chain-id lava-testnet-2
lavad init "$NODE_MONIKER" --chain-id lava-testnet-2

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/lava/genesis.json > $HOME/.lava/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/lava/addrbook.json > $HOME/.lava/config/addrbook.json

APP_TOML="~/.lava/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $APP_TOML

CONFIG_TOML="~/.lava/config/config.toml"
SEEDS="3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025ulava"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 30/g' $CONFIG_TOML
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 30/g' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/lavad.service > /dev/null << EOF
[Unit]
Description=Lava Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
URL="https://snapshots-testnet.stake-town.com/lava/lava-testnet-2_latest.tar.lz4"
curl -L $URL | tar -Ilz4 -xf - -C $HOME/.lava
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.lava/config/config.toml"
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
sudo systemctl enable lavad
sudo systemctl start lavad

sudo journalctl -u lavad -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.lava/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.lava/config/config.toml
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
lavad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
lavad keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
lavad tx staking create-validator 
--amount=1000000ulava \
--pubkey=$(lavad tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=lava-testnet-2 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1ulava \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
