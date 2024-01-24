---
cover: ../../.gitbook/assets/andromeda-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/andromeda/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/andromedad
git clone https://github.com/andromedaprotocol/andromedad.git
cd $HOME/andromedad || return
git checkout andromeda-1-v0.1.0

make install

andromedad config keyring-backend os
andromedad config chain-id andromeda-1
andromedad init "Your Moniker" --chain-id andromeda-1

# Download genesis and addrbook
curl -Ls https://snapshots.stake-town.com/andromeda/genesis.json > $HOME/.andromeda/config/genesis.json
curl -Ls https://snapshots.stake-town.com/andromeda/addrbook.json > $HOME/.andromeda/config/addrbook.json

APP_TOML="~/.andromeda/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.andromeda/config/config.toml"
SEEDS=""
PEERS="28876b3094518bef97a1250ef641c26b7d4a658d@138.201.21.197:39656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uandr"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.andromeda/cosmovisor/genesis/bin
mkdir -p ~/.andromeda/cosmovisor/upgrades
cp ~/go/bin/andromedad ~/.andromeda/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/andromedad.service > /dev/null << EOF
[Unit]
Description=Andromeda Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=andromedad"
Environment="DAEMON_HOME=$HOME/.andromeda"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
andromedad tendermint unsafe-reset-all --home $HOME/.andromeda --keep-addr-book

URL=https://snapshots.stake-town.com/andromeda/andromeda-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.andromeda
[[ -f $HOME/.andromeda/data/upgrade-info.json ]] && cp $HOME/.andromeda/data/upgrade-info.json $HOME/.andromeda/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.andromeda/config/config.toml"
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
sudo systemctl enable andromedad
sudo systemctl start andromedad

sudo journalctl -u andromedad -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.andromeda/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.andromeda/config/config.toml
sudo systemctl restart andromedad && sudo journalctl -u andromedad -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
andromedad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
andromedad keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
andromedad tx staking create-validator \
--amount=1000000uandr \
--pubkey=$(andromedad tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=andromeda-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.001uandr \
--gas-adjustment=1.4 \
--gas=auto \
-y
```
