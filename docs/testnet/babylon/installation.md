---
cover: ../../.gitbook/assets/babylon banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/babylon/install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd || return
rm -rf babylon
git clone https://github.com/babylonchain/babylon
cd babylon || return
git checkout v0.7.2
make install
babylond version # v0.7.2

babylond config keyring-backend test
babylond config chain-id bbn-test-2
babylond init "<Your moniker>" --chain-id bbn-test-2

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/babylon/genesis.json > $HOME/.babylond/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/babylon/addrbook.json > $HOME/.babylond/config/addrbook.json

APP_TOML="~/.babylond/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $APP_TOML
sed -i 's|^network *=.*|network = "mainnet"|g' $APP_TOML

CONFIG_TOML="~/.babylond/config/config.toml"
SEEDS="03ce5e1b5be3c9a81517d415f65378943996c864@18.207.168.204:26656,a5fabac19c732bf7d814cf22e7ffc23113dc9606@34.238.169.221:26656"
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.00001ubbn"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i 's|^timeout_commit *=.*|timeout_commit = "10s"|g' $CONFIG_TOML
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 30/g' $CONFIG_TOML
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 30/g' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/babylond.service > /dev/null << EOF
[Unit]
Description=Babylon Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which babylond) start --x-crisis-skip-assert-invariants
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
URL="https://snapshots-testnet.stake-town.com/babylon/bbn-test-2_latest.tar.lz4"
curl -L $URL | tar -Ilz4 -xf - -C $HOME/.babylond
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.babylond/config/config.toml"
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

First, we need to create BLS record

```bash
babylond create-bls-key $(babylond keys show <YOUR_WALLET_NAME> -a)
sudo systemctl restart babylond
```

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
babylond tx checkpointing create-validator \
--amount=1000000ubbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=bbn-test-2 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000ubbn
-y
```
