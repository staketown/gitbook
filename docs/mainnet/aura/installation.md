---
cover: ../../.gitbook/assets/aura-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/aura/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf aura
git clone https://github.com/aura-nw/aura.git
cd aura || return
git checkout aura_v0.4.5

make install

aurad config keyring-backend os
aurad config chain-id xstaxy-1
aurad init "Your Moniker" --chain-id xstaxy-1

# Download genesis and addrbook
curl -Ls https://snapshots.stake-town.com/aura/genesis.json > $HOME/.aura/config/genesis.json
curl -Ls https://snapshots.stake-town.com/aura/addrbook.json > $HOME/.aura/config/addrbook.json

APP_TOML="~/.aura/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.aura/config/config.toml"
SEEDS="22a0ca5f64187bb477be1d82166b1e9e184afe50@18.143.52.13:26656,0b8bd8c1b956b441f036e71df3a4d96e85f843b8@13.250.159.219:26656"
PEERS="2bd24dbac94c3b95707c8f06a57d61a31ae666fa@88.99.208.54:46656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uaura"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.aura/cosmovisor/genesis/bin
mkdir -p ~/.aura/cosmovisor/upgrades
cp ~/go/bin/aurad ~/.aura/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/aurad.service > /dev/null << EOF
[Unit]
Description=Aura Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=aurad"
Environment="DAEMON_HOME=$HOME/.aura"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
aurad tendermint unsafe-reset-all --home $HOME/.aura --keep-addr-book

URL=https://snapshots.stake-town.com/aura/xstaxy-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.aura
[[ -f $HOME/.aura/data/upgrade-info.json ]] && cp $HOME/.aura/data/upgrade-info.json $HOME/.aura/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.aura/config/config.toml"
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
sudo systemctl enable aurad
sudo systemctl start aurad

sudo journalctl -u aurad -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.aura/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.aura/config/config.toml
sudo systemctl restart aurad && sudo journalctl -u aurad -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
aurad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
aurad keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
aurad tx staking create-validator \
--amount=1000000uaura \
--pubkey=$(aurad tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=xstaxy-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.001uaura \
--gas-adjustment=1.4\
--gas=auto \
-y
```
