---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/c4e/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/c4e-chain
git clone https://github.com/chain4energy/c4e-chain.git
cd $HOME/c4e-chain || return
git checkout v1.4.0

make install

c4ed config keyring-backend os
c4ed config chain-id babajaga-1
c4ed init "Your Moniker" --chain-id babajaga-1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/c4e/genesis.json > $HOME/.c4e-chain/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/c4e/addrbook.json > $HOME/.c4e-chain/config/addrbook.json

APP_TOML="~/.c4e-chain/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.c4e-chain/config/config.toml"
SEEDS=""
PEERS="94a46ce2a5c5b835b84e121676847c5ee4eabf3f@65.109.65.248:35656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0025uc4e"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.c4e-chain/cosmovisor/genesis/bin
mkdir -p ~/.c4e-chain/cosmovisor/upgrades
cp ~/go/bin/c4ed ~/.c4e-chain/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/c4ed.service > /dev/null << EOF
[Unit]
Description=C4E Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=c4ed"
Environment="DAEMON_HOME=$HOME/.c4e-chain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/c4e/babajaga-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.c4e-chain
[[ -f $HOME/.c4e-chain/data/upgrade-info.json ]] && cp $HOME/.c4e-chain/data/upgrade-info.json $HOME/.c4e-chain/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.c4e-chain/config/config.toml"
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
sudo systemctl enable c4ed
sudo systemctl start c4ed

sudo journalctl -u c4ed -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.c4e-chain/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.c4e-chain/config/config.toml
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
c4ed keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
c4ed keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
c4ed tx staking create-validator \
--amount=1000000uc4e \
--pubkey=$(c4ed tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=babajaga-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1uc4e \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
