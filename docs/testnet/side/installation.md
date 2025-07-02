---
cover: ../../.gitbook/assets/side-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/side/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/side
git clone https://github.com/sideprotocol/side.git
cd $HOME/side || return
git checkout v2.0.0-rc.6

make install

sided config keyring-backend os
sided config chain-id sidechain-testnet-6
sided init "Your Moniker" --chain-id sidechain-testnet-6

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/side/genesis.json > $HOME/.side/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/side/addrbook.json > $HOME/.side/config/addrbook.json

APP_TOML="~/.side/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.side/config/config.toml"
SEEDS=""
PEERS="e1534b92058cf04e83ee9b2226272d2587510025@78.46.156.187:26656,e5b1682f3783f73ecc5e2bf50ef2104804108fde@88.198.70.23:26356,430ffeebf52328322ec5f8470b260ea9a5469c25@205.209.125.118:60656,e953d5ac69f7ef9ca8608b7ef8c600a8b4663d44@195.201.241.107:56146,fc350bf644f03278df11b8735727cc2ead4134c9@65.109.93.152:26786,c02ea49a61e386c2442771c795faa1847d631661@141.94.143.203:56146,ac943e8a90d12ee061f2f0109b9e1849c0be90e5@213.239.198.181:36656,8fe46ca180a5c6989e83de31755e6e21f6de67f3@80.240.21.182:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.005uside"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.side/cosmovisor/genesis/bin
mkdir -p ~/.side/cosmovisor/upgrades
cp ~/go/bin/sided ~/.side/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/sided.service > /dev/null << EOF
[Unit]
Description=Side Protocol Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=sided"
Environment="DAEMON_HOME=$HOME/.side"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
sided tendermint unsafe-reset-all --home $HOME/.side --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/side/sidechain-testnet-6_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.side
[[ -f $HOME/.side/data/upgrade-info.json ]] && cp $HOME/.side/data/upgrade-info.json $HOME/.side/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.side/config/config.toml"
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
sudo systemctl enable sided
sudo systemctl start sided

sudo journalctl -u sided -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.side/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.side/config/config.toml
sudo systemctl restart sided && sudo journalctl -u sided -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
sided keys add <YOUR_WALLET_NAME> --key-type="segwit"
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
sided keys add <YOUR_WALLET_NAME> --recover --key-type="segwit"
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
sided tx staking create-validator \
--amount=1000000uside \
--pubkey=$(sided tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=sidechain-testnet-6 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000uside \
-y
```
