---
cover: ../../.gitbook/assets/evmos-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/evmos/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf evmos
git clone https://github.com/tharsis/evmos
cd evmos || return
git checkout v16.0.0

make install

evmosd config keyring-backend os
evmosd config chain-id evmos_9001-2
evmosd init "<Your moniker>" --chain-id evmos_9001-2

curl -s https://snapshots.polkachu.com/genesis/evmos/genesis.json > $HOME/.evmosd/config/genesis.json
curl -s https://snapshots.polkachu.com/addrbook/evmos/addrbook.json > $HOME/.evmosd/config/addrbook.json

APP_TOML=$HOME/.evmosd/config/app.toml
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "19"|g' $APP_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "250000000aevmos"|g' $APP_TOML

CONFIG_TOML=$HOME/.evmosd/config/config.toml
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
SEEDS="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:13456"
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML

# Install and configure cosmovisor...

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.evmosd/cosmovisor/genesis/bin
mkdir -p ~/.evmosd/cosmovisor/upgrades
cp ~/go/bin/evmosd ~/.evmosd/cosmovisor/genesis/bin

# Starting service and synchronization...

sudo tee /etc/systemd/system/evmosd.service > /dev/null << EOF
[Unit]
Description=Evmos Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=evmosd"
Environment="DAEMON_HOME=$HOME/.evmosd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book

# Add snapshot here
SNAP_NAME=$(curl -s https://ss.evmos.nodestake.top/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss.evmos.nodestake.top/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.evmosd
[[ -f $HOME/.evmosd/data/upgrade-info.json ]]  && cp $HOME/.evmosd/data/upgrade-info.json $HOME/.evmosd/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.evmosd/config/config.toml"
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
sudo systemctl enable evmosd
sudo systemctl start evmosd

sudo journalctl -u evmosd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.evmosd/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.evmosd/config/config.toml
systemctl restart evmosd && journalctl -u evmosd -f -o cat
```

## Wallet creation

Create wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
evmosd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
evmosd keys add <YOUR_WALLET_NAME> --recover
```

## Validator creation

After successful synchronisation we can proceed with validation creation.

```bash
evmosd tx staking create-validator \
--amount=1000000000000000000aevmos \
--pubkey=$(evmosd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=evmos_9001-2 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.001aevmos \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
