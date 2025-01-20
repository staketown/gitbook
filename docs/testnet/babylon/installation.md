---
cover: ../../.gitbook/assets/babylon-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/babylon/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/babylon
git clone https://github.com/babylonlabs-io/babylon.git
cd $HOME/babylon || return
git checkout v1.0.0-rc.3

make install

babylond config keyring-backend os
babylond config chain-id bbn-test-5
babylond init "Your Moniker" --chain-id bbn-test-5

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/babylon/genesis.json > $HOME/.babylond/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/babylon/addrbook.json > $HOME/.babylond/config/addrbook.json

APP_TOML="~/.babylond/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.babylond/config/config.toml"
SEEDS=""
PEERS="e0a8511a01f33de0dbc02b7b0cc53b3815fb3d7a@65.108.203.61:48656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.005ubbn"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.babylond/cosmovisor/genesis/bin
mkdir -p ~/.babylond/cosmovisor/upgrades
cp ~/go/bin/babylond ~/.babylond/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/babylond.service > /dev/null << EOF
[Unit]
Description=Babylon Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=babylond"
Environment="DAEMON_HOME=$HOME/.babylond"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
babylond tendermint unsafe-reset-all --home $HOME/.babylond --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/babylon/bbn-test-5_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.babylond
[[ -f $HOME/.babylond/data/upgrade-info.json ]] && cp $HOME/.babylond/data/upgrade-info.json $HOME/.babylond/cosmovisor/genesis/upgrade-info.json
```

Enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable babylond
sudo systemctl start babylond

sudo journalctl -u babylond -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.babylond/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.babylond/config/config.toml
sudo systemctl restart babylond && sudo journalctl -u babylond -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
babylond keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
babylond keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
babylond tx staking create-validator \
--amount=1000000uabbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=bbn-test-5 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.025ubbn \
--gas-adjustment=1.6 \
--gas=auto \
-y
```
