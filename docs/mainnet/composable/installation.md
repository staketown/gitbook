---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Installation

Automatic

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/composable/main_install.sh)
```

Manual

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-centauri.git
cd $HOME/composable-testnet || return
git checkout v6.4.2

make install

centaurid config keyring-backend file
centaurid config chain-id centauri-1
centaurid init "<Your moniker>" --chain-id centauri-1

# Download genesis and addrbook
curl -s https://snapshots.stake-town.com/composable/genesis.json > $HOME/.banksy/config/genesis.json
curl -s https://snapshots.stake-town.com/composable/addrbook.json > $HOME/.banksy/config/addrbook.json

APP_TOML="~/.banksy/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $APP_TOML

CONFIG_TOML="~/.banksy/config/config.toml"
SEEDS="364b8245e72f083b0aa3e0d59b832020b66e9e9d@65.109.80.150:21500"
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0ppica"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 30/g' $CONFIG_TOML
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 30/g' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/centaurid.service > /dev/null << EOF
[Unit]
Description=Composable Node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME
ExecStart=$(which centaurid) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
URL="https://snapshots.stake-town.com/composable/centauri-1_latest.tar.lz4"
curl $URL | lz4 -dc - | tar -xf - -C $HOME/.banksy
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.banksy/config/config.toml"
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
sudo systemctl enable centaurid
sudo systemctl start centaurid

sudo journalctl -u centaurid -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.banksy/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.banksy/config/config.toml
sudo systemctl restart banksyd && sudo journalctl -u banksyd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
centaurid keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
centaurid keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
centaurid tx staking create-validator \
--amount=1000000ppica \
--pubkey=$(centaurid tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1ppica \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
