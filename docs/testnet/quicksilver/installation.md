---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/quicksilver/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
wget -O quicksilverd https://github.com/quicksilver-zone/quicksilver/releases/download/v1.6.0-beta1/quicksilverd-v1.6.0-beta1-amd64
chmod +x quicksilverd
mv quicksilverd $HOME/go/bin

quicksilverd config keyring-backend os
quicksilverd config chain-id rhye-2
quicksilverd init "Your Moniker" --chain-id rhye-2

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/quicksilver/genesis.json > $HOME/.quicksilverd/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/quicksilver/addrbook.json > $HOME/.quicksilverd/config/addrbook.json

APP_TOML="~/.quicksilverd/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.quicksilverd/config/config.toml"
SEEDS=""
PEERS="5fc67b60aff6ce69e7b183cb35d045add8f3cf8e@65.109.65.248:50656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uqck"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.quicksilverd/cosmovisor/genesis/bin
mkdir -p ~/.quicksilverd/cosmovisor/upgrades
cp ~/go/bin/quicksilverd ~/.quicksilverd/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/quicksilverd.service > /dev/null << EOF
[Unit]
Description=Quicksilver Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=quicksilverd"
Environment="DAEMON_HOME=$HOME/.quicksilverd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
quicksilverd tendermint unsafe-reset-all --home $HOME/.quicksilverd --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/quicksilver/rhye-2_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.quicksilverd
[[ -f $HOME/.quicksilverd/data/upgrade-info.json ]] && cp $HOME/.quicksilverd/data/upgrade-info.json $HOME/.quicksilverd/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.quicksilverd/config/config.toml"
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
sudo systemctl enable quicksilverd
sudo systemctl start quicksilverd

sudo journalctl -u quicksilverd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.quicksilverd/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.quicksilverd/config/config.toml
sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
quicksilverd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
quicksilverd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
quicksilverd tx staking create-validator \
--amount=1000000uqck \
--pubkey=$(quicksilverd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=rhye-2 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.0001uqck \
--gas-adjustment=1.4 \
--gas=auto \
-y
```
