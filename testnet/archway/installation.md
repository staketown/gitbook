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
rm -rf archway
git clone https://github.com/archway-network/archway.git
cd archway || return
git checkout v4.0.0
make install

archwayd version # v4.0.0

archwayd config keyring-backend os
archwayd config chain-id constantine-3
archwayd init "<Your moniker>" --chain-id constantine-3

curl -s https://snapshots-testnet.stake-town.com/archway/genesis.json > $HOME/.archway/config/genesis.json
curl -s https://snapshots-testnet.stake-town.com/archway/addrbook.json > $HOME/.archway/config/addrbook.json

APP_TOML=$HOME/.archway/config/app.toml
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "19"|g' $APP_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0aconst"|g' $APP_TOML

CONFIG_TOML=$HOME/.archway/config/config.toml
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
SEEDS="3f472746f46493309650e5a033076689996c8881@archway-testnet.rpc.kjnodes.com:15659"
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/archwayd.service > /dev/null << EOF
[Unit]
Description=Archway Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which archwayd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

archwayd tendermint unsafe-reset-all --home $HOME/.archway --keep-addr-book

# Add snapshot here
URL="https://snapshots-testnet.stake-town.com/archway/constantine-3_latest.tar.lz4"
curl $URL | lz4 -dc - | tar -xf - -C $HOME/.archway
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
systemctl restart archwayd && journalctl -u archwayd -f -o cat
```

## Wallet creation

Create wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
archwayd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
archwayd keys add <YOUR_WALLET_NAME> --recover
```

## Validator creation

After successful synchronisation we can proceed with validation creation.

```bash
archwayd tx staking create-validator \
--amount=1000000aconst \
--pubkey=$(archwayd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=constantine-3 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1aconst \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
