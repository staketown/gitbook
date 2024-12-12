---
cover: ../../.gitbook/assets/{{BANNER_NAME}}
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/{{SCRIPT_DIR}}/{{SCRIPT_NAME}})
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf {{PROJECT_DIR}}
git clone {{PROJECT_GIT_URL}}
cd {{PROJECT_DIR}} || return
git checkout {{BINARY_VERSION}}

make install

{{BINARY}} config keyring-backend os
{{BINARY}} config chain-id {{CHAIN_ID}}
{{BINARY}} init "Your Moniker" --chain-id {{CHAIN_ID}}

# Download genesis and addrbook
curl -Ls https://snapshots{{SNAP_SUBDIR}}.stake-town.com/{{SCRIPT_DIR}}/genesis.json > $HOME/{{WORKING_DIR}}/config/genesis.json
curl -Ls https://snapshots{{SNAP_SUBDIR}}.stake-town.com/{{SCRIPT_DIR}}/addrbook.json > $HOME/{{WORKING_DIR}}/config/addrbook.json

APP_TOML="~/{{WORKING_DIR}}/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 19|g' $APP_TOML

CONFIG_TOML="~/{{WORKING_DIR}}/config/config.toml"
SEEDS="{{SEEDS}}"
PEERS="{{PEERS}}"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "{{MINIMUM_GAS_PRICES}}"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/{{WORKING_DIR}}/cosmovisor/genesis/bin
mkdir -p ~/{{WORKING_DIR}}/cosmovisor/upgrades
cp ~/go/bin/{{BINARY}} ~/{{WORKING_DIR}}/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/{{BINARY}}.service > /dev/null << EOF
[Unit]
Description={{PROJECT}} Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME={{BINARY}}"
Environment="DAEMON_HOME=$HOME/{{WORKING_DIR}}"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
{{BINARY}} tendermint unsafe-reset-all --home $HOME/{{WORKING_DIR}} --keep-addr-book

URL=https://snapshots{{SNAP_SUBDIR}}.stake-town.com/{{SCRIPT_DIR}}/{{CHAIN_ID}}_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/{{WORKING_DIR}}
[[ -f $HOME/{{WORKING_DIR}}/data/upgrade-info.json ]] && cp $HOME/{{WORKING_DIR}}/data/upgrade-info.json $HOME/{{WORKING_DIR}}/cosmovisor/genesis/upgrade-info.json
```

Enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable {{BINARY}}
sudo systemctl start {{BINARY}}

sudo journalctl -u {{BINARY}} -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/{{WORKING_DIR}}/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/{{WORKING_DIR}}/config/config.toml
sudo systemctl restart {{BINARY}} && sudo journalctl -u {{BINARY}} -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
{{BINARY}} keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
{{BINARY}} keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
{{BINARY}} tx staking create-validator \
--amount={{AMOUNT}} \
--pubkey=$({{BINARY}} tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id={{CHAIN_ID}} \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices={{GAS_PRICES}} \
--gas-adjustment={{GAS_ADJUSTMENT}} \
--gas=auto \
-y
```

### Price feeder installation

> ⚠️ save your mnemonic during creation wallet for feeder, you will find it in logs

You must have some deposit on your main wallet just to send some token to created price feeder wallet.

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/{{SCRIPT_DIR}}/price_feeder_install.sh)
```

### Remove price feeder

> ⚠️ removing price feeder and wallet created for it

```bash
sudo systemctl disable {{SCRIPT_DIR}}-price-feeder.service && \
sudo systemctl daemon-reload && \
sudo rm /etc/systemd/system/{{SCRIPT_DIR}}-price-feeder.service && \
sudo rm -rf $HOME/price-feeder && \
sudo rm -rf $HOME/.{{SCRIPT_DIR}}-price-feeder && \
umeed keys delete price_feeder_wallet
```
