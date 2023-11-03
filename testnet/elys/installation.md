---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Installation

Automatic

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/elys/install.sh)
```

Manual

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf elys
URL="https://github.com/elys-network/elys/releases/download/v0.7.0/elys._v0.7.0_linux_amd64.tar.gz"
wget $URL | tar -xf elys._v0.7.0_linux_amd64.tar.gz -C $HOME/go/bin

elysd version # v0.12.0

elysd config keyring-backend os
elysd config chain-id elystestnet-1
elysd init "<Your moniker>" --chain-id elystestnet-1

# Download genesis and addrbook
curl -s https://snapshots-testnet.stake-town.com/elys/genesis.json > $HOME/.elys/config/genesis.json
curl -s https://snapshots-testnet.stake-town.com/elys/addrbook.json > $HOME/.elys/config/addrbook.json

APP_TOML="~/.elys/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $APP_TOML

CONFIG_TOML="~/.elys/config/config.toml"
SEEDS=""
PEERS="b651ea2a0517e82c1a476e25966ab3de3159afe8@34.229.22.39:26656,3b389873f999763d3f937f63f765f0948411e296@44.192.85.92:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0018ibc/2180E84E20F5679FCC760D8C165B60F42065DEF7F46A72B447CFF1B7DC6C0A65,0.00025ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2,0.00025uelys"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 30/g' $CONFIG_TOML
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 30/g' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/elysd.service > /dev/null << EOF
[Unit]
Description=Elys Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which elysd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

elysd tendermint unsafe-reset-all --home $HOME/.elys --keep-addr-book

# Add snapshot here
URL="https://snapshots-testnet.stake-town.com/elys/elystestnet-1_latest.tar.lz4"
curl $URL | lz4 -dc - | tar -xf - -C $HOME/.elys
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.elys/config/config.toml"
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
sudo systemctl enable elysd
sudo systemctl start elysd

sudo journalctl -u elysd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.elys/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.elys/config/config.toml
sudo systemctl restart elysd && sudo journalctl -u elysd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
elysd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
elysd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
elysd tx staking create-validator \
--amount=1000000uelys \
--pubkey=$(elysd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=elystestnet-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1uelys \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
