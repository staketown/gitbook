---
cover: ../../.gitbook/assets/elys-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/elys/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/elys
git clone https://github.com/elys-network/elys.git
cd $HOME/elys || return
git checkout v1.6.0

make install

elysd config keyring-backend os
elysd config chain-id elysicstestnet-1
elysd init "Your Moniker" --chain-id elysicstestnet-1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/elys/genesis.json > $HOME/.elys/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/elys/addrbook.json > $HOME/.elys/config/addrbook.json

APP_TOML="~/.elys/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.elys/config/config.toml"
SEEDS=""
PEERS="b64643dc38d426362a4f7c98b6acabe37ffb5654@65.108.203.61:38656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0003uelys,0.001ibc/F082B65C88E4B6D5EF1DB243CDA1D331D002759E938A0F5CD3FFDC5D53B3E349,0.001ibc/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.elys/cosmovisor/genesis/bin
mkdir -p ~/.elys/cosmovisor/upgrades
cp ~/go/bin/elysd ~/.elys/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/elysd.service > /dev/null << EOF
[Unit]
Description=Elys Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=elysd"
Environment="DAEMON_HOME=$HOME/.elys"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
elysd tendermint unsafe-reset-all --home $HOME/.elys --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/elys/elysicstestnet-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.elys
[[ -f $HOME/.elys/data/upgrade-info.json ]] && cp $HOME/.elys/data/upgrade-info.json $HOME/.elys/cosmovisor/genesis/upgrade-info.json
```

Enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable elysd
sudo systemctl start elysd

sudo journalctl -u elysd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.elys/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.elys/config/config.toml
sudo systemctl restart elysd && sudo journalctl -u elysd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
elysd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
elysd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
elysd tx staking create-validator \
--amount=1000000uelys \
--pubkey=$(elysd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=elysicstestnet-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.003uelys \
--gas-adjustment=1.4 \
--gas=auto \
-y
```
