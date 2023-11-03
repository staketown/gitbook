---
cover: ../../.gitbook/assets/cascadia-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/cascadia/install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd || return
rm -rf cascadia
git clone https://github.com/cascadiafoundation/cascadia
cd $HOME/cascadia || return
git checkout v0.1.7
make install
cascadiad version # v0.1.7

cascadiad config keyring-backend os
cascadiad config chain-id cascadia_6102-1
cascadiad init "<Your moniker>" --chain-id cascadia_6102-1

curl -s https://snapshots-testnet.stake-town.com/cascadia/genesis.json > $HOME/.cascadiad/config/genesis.json
curl -s https://snapshots-testnet.stake-town.com/cascadia/addrbook.json > $HOME/.cascadiad/config/addrbook.json

APP_TOML=$HOME/.cascadiad/config/app.toml
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "19"|g' $APP_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0025aCC"|g' $APP_TOML

CONFIG_TOML=$HOME/.cascadiad/config/config.toml
PEERS="b651ea2a0517e82c1a476e25966ab3de3159afe8@34.229.22.39:26656,3b389873f999763d3f937f63f765f0948411e296@44.192.85.92:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
SEEDS=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML

# Install and configure cosmovisor...

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.cascadiad/cosmovisor/genesis/bin
mkdir -p ~/.cascadiad/cosmovisor/upgrades
cp ~/go/bin/cascadiad $HOME/.cascadiad/cosmovisor/genesis/bin

# Starting service and synchronization...

sudo tee /etc/systemd/system/cascadiad.service > /dev/null << EOF
[Unit]
Description=Cascadia Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --chain-id cascadia_6102-1
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=cascadiad"
Environment="DAEMON_HOME=$HOME/.cascadiad"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

cascadiad tendermint unsafe-reset-all --home $HOME/.cascadiad --keep-addr-book

# Add snapshot here
URL="https://snapshots-testnet.stake-town.com/cascadia/cascadia_6102-1_latest.tar.lz4"
curl $URL | lz4 -dc - | tar -xf - -C $HOME/.cascadiad
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.cascadiad/config/config.toml"
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
sudo systemctl enable cascadiad
sudo systemctl start cascadiad

sudo journalctl -u cascadiad -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.cascadiad/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.cascadiad/config/config.toml
systemctl restart cascadiad && journalctl -u cascadiad -f -o cat
```

## Wallet creation

Create wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
cascadiad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
cascadiad keys add <YOUR_WALLET_NAME> --recover
```

## Validator creation

After successful synchronisation we can proceed with validation creation.

```bash
cascadiad tx staking create-validator \
--amount=1000000aCC \
--pubkey=$(cascadiad tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=cascadia_6102-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1aCC \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
