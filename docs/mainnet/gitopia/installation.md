---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/gitopia/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/gitopia
git clone https://github.com/gitopia/gitopia.git
cd $HOME/gitopia || return
git checkout v5.1.0

make install

gitopiad config keyring-backend os
gitopiad config chain-id gitopia
gitopiad init "Your Moniker" --chain-id gitopia

# Download genesis and addrbook
curl -Ls https://snapshots.stake-town.com/gitopia/genesis.json > $HOME/.gitopia/config/genesis.json
curl -Ls https://snapshots.stake-town.com/gitopia/addrbook.json > $HOME/.gitopia/config/addrbook.json

APP_TOML="~/.gitopia/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.gitopia/config/config.toml"
SEEDS=""
PEERS="d5525675ceb88d2c4f4df828ec01d237bcc11950@138.201.21.197:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001ulore"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.gitopia/cosmovisor/genesis/bin
mkdir -p ~/.gitopia/cosmovisor/upgrades
cp ~/go/bin/gitopiad ~/.gitopia/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/gitopiad.service > /dev/null << EOF
[Unit]
Description=Gitopia Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=gitopiad"
Environment="DAEMON_HOME=$HOME/.gitopia"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia --keep-addr-book

URL=https://snapshots.stake-town.com/gitopia/gitopia_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.gitopia
[[ -f $HOME/.gitopia/data/upgrade-info.json ]] && cp $HOME/.gitopia/data/upgrade-info.json $HOME/.gitopia/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.gitopia/config/config.toml"
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
sudo systemctl enable gitopiad
sudo systemctl start gitopiad

sudo journalctl -u gitopiad -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.gitopia/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.gitopia/config/config.toml
sudo systemctl restart gitopiad && sudo journalctl -u gitopiad -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
gitopiad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
gitopiad keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
gitopiad tx staking create-validator \
--amount=1000000ulore \
--pubkey=$(gitopiad tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=gitopia \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1ulore \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
