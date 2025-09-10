---
cover: ../../.gitbook/assets/bitway-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/bitway/main_install.sh)
```

Manual installation 

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return 
rm -rf $HOME/bitway
git clone https://github.com/bitwaylabs/bitway.git
cd $HOME/bitway || return
git checkout v2.0.0

make install

bitwayd config keyring-backend os
bitwayd config chain-id bitway-1
bitwayd init "Your Moniker" --chain-id bitwayd-1

# Download genesis and addrbook
curl -Ls https://snapshots-1.stake-town.com/bitway/genesis.json > $HOME/.bitway/config/genesis.json
curl -Ls https://snapshots-1.stake-town.com/bitway/addrbook.json > $HOME/.bitway/config/addrbook.json

APP_TOML="~/.bitway/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.bitway/config/config.toml"
SEEDS="3f472746f46493309650e5a033076689996c8881@bitway.rpc.kjnodes.com:17459"
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0006ubtw,0.000001sat"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.bitway/cosmovisor/genesis/bin
mkdir -p ~/.bitway/cosmovisor/upgrades
cp ~/go/bin/bitwayd ~/.bitway/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/bitwayd.service > /dev/null << EOF
[Unit]
Description=Bitway Protocol Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=bitwayd"
Environment="DAEMON_HOME=$HOME/.bitway"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
bitwayd tendermint unsafe-reset-all --home $HOME/.bitway --keep-addr-book

URL=https://snapshots-1.stake-town.com/bitway/bitway-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.bitway
[[ -f $HOME/.bitway/data/upgrade-info.json ]] && cp $HOME/.bitway/data/upgrade-info.json $HOME/.bitway/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.bitway/config/config.toml"
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
sudo systemctl enable bitwayd
sudo systemctl start bitwayd

sudo journalctl -u bitwayd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.bitway/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.bitway/config/config.toml
sudo systemctl restart bitwayd && sudo journalctl -u bitwayd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
bitwayd keys add <YOUR_WALLET_NAME> --key-type="segwit"
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
bitwayd keys add <YOUR_WALLET_NAME> --recover --key-type="segwit"
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
bitwayd tx staking create-validator \
--amount=1000000ubtw \
--pubkey=$(bitwayd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=bitway-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000ubtw \
-y
```
