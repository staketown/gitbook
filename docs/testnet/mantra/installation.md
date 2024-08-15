---
cover: ../../.gitbook/assets/mantra-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/mantra/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
sudo wget -O /usr/lib/libwasmvm.x86_64.so https://github.com/CosmWasm/wasmvm/releases/download/v1.3.1/libwasmvm.x86_64.so
wget https://github.com/MANTRA-Finance/public/releases/download/v3.0.0/mantrachaind-3.0.0-linux-amd64.tar.gz
tar -xvf mantrachaind-3.0.0-linux-amd64.tar.gz
rm mantrachaind-3.0.0-linux-amd64.tar.gz
mv mantrachaind $HOME/go/bin

mantrachaind config keyring-backend os
mantrachaind config chain-id mantra-hongbai-1
mantrachaind init "Your Moniker" --chain-id mantra-hongbai-1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/mantra/genesis.json > $HOME/.mantrachain/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/mantra/addrbook.json > $HOME/.mantrachain/config/addrbook.json

APP_TOML="~/.mantrachain/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.mantrachain/config/config.toml"
SEEDS=""
PEERS="eaeb4872c88fa5d3a3fea9deb93cecb552dbe7a3@65.109.65.248:47656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0002uom"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.mantrachain/cosmovisor/genesis/bin
mkdir -p ~/.mantrachain/cosmovisor/upgrades
cp ~/go/bin/mantrachaind ~/.mantrachain/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/mantrachaind.service > /dev/null << EOF
[Unit]
Description=Mantra Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=mantrachaind"
Environment="DAEMON_HOME=$HOME/.mantrachain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
mantrachaind tendermint unsafe-reset-all --home $HOME/.mantrachain --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/mantra/mantra-hongbai-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.mantrachain
[[ -f $HOME/.mantrachain/data/upgrade-info.json ]] && cp $HOME/.mantrachain/data/upgrade-info.json $HOME/.mantrachain/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.mantrachain/config/config.toml"
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
sudo systemctl enable mantrachaind
sudo systemctl start mantrachaind

sudo journalctl -u mantrachaind -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.mantrachain/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.mantrachain/config/config.toml
sudo systemctl restart mantrachaind && sudo journalctl -u mantrachaind -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
mantrachaind keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
mantrachaind keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
mantrachaind tx staking create-validator \
--amount=1000000uom \
--pubkey=$(mantrachaind tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=mantra-hongbai-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.0002uom \
--gas-adjustment=1.7 \
--gas=auto \
-y
```
