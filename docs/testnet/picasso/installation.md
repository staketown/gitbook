---
cover: ../../.gitbook/assets/picasso-banner.jpeg
coverY: 0
---

# Installation

Automatic

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/picasso/install.sh)
```

Manual

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf composable-testnet
git clone https://github.com/notional-labs/composable-testnet.git
cd $HOME/composable-testnet || return
git checkout v7.0.1-devnet-rc2

make install

picad config keyring-backend os
picad config chain-id banksy-testnet-5
picad init "<Your moniker>" --chain-id banksy-testnet-5

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/picasso/genesis.json > $HOME/.banksy/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/picasso/addrbook.json > $HOME/.banksy/config/addrbook.json

APP_TOML="~/.banksy/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $APP_TOML

CONFIG_TOML="~/.banksy/config/config.toml"
SEEDS="6e8a56df9b9c52a730dd780172fc135a96a9feda@65.109.26.223:26656"
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

# Install and configure cosmovisor...

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.banksy/cosmovisor/genesis/bin
mkdir -p ~/.banksy/cosmovisor/upgrades
cp ~/go/bin/picad $HOME/.banksy/cosmovisor/genesis/bin

# Starting service and synchronization...

sudo tee /etc/systemd/system/picad.service > /dev/null << EOF
[Unit]
Description=Picasso Node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=picad"
Environment="DAEMON_HOME=$HOME/.banksy"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

picad tendermint unsafe-reset-all --home $HOME/.banksy --keep-addr-book

# Snapshots
URL=https://snapshots-testnet.stake-town.com/picasso/banksy-testnet-5_latest.tar.lz4
curl -L $URL | tar -Ilz4 -xf - -C $HOME/.banksy
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
sudo systemctl enable picad
sudo systemctl start picad

sudo journalctl -u picad -f -o cat
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
picad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
picad keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
picad tx staking create-validator \
--amount=1000000ppica \
--pubkey=$(picad tendermint show-validator) \
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
