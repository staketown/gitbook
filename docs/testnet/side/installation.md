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
rm -rf sidechain
git clone https://github.com/sideprotocol/sidechain.git
cd sidechain || return
git checkout v0.8.0

make install

sided config keyring-backend os
sided config chain-id S2-testnet-1
sided init "Your Moniker" --chain-id S2-testnet-1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/side/genesis.json > $HOME/.side/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/side/addrbook.json > $HOME/.side/config/addrbook.json

APP_TOML="~/.side/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.side/config/config.toml"
SEEDS=""
PEERS="027ef6300590b1ca3a2b92a274247e24537bd9c9@65.109.65.248:49656"
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

URL=https://snapshots-testnet.stake-town.com/side/S2-testnet-1_latest.tar.lz4
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
sided keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
sided keys add <YOUR_WALLET_NAME> --recover
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
--chain-id=S2-testnet-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000uside \
-y
```

### Pricefeeder installation

Install pricefeeder binary

```shell
curl -s https://get.nibiru.fi/pricefeeder@v0.21.6! | bash
```

Set variables for pricefeeder

```shell
export CHAIN_ID="S2-testnet-1"
export GRPC_ENDPOINT="localhost:9090"
export WEBSOCKET_ENDPOINT="ws://localhost:26657/websocket"
export EXCHANGE_SYMBOLS_MAP='{"bitfinex":{"ubtc:unusd":"tBTCUSD","ubtc:uusd":"tBTCUSD","ueth:unusd":"tETHUSD","ueth:uusd":"tETHUSD","uusdc:uusd":"tUDCUSD","uusdc:unusd":"tUDCUSD"},"coingecko":{"ubtc:uusd":"bitcoin","ubtc:unusd":"bitcoin","ueth:uusd":"ethereum","ueth:unusd":"ethereum","uusdt:uusd":"tether","uusdt:unusd":"tether","uusdc:uusd":"usd-coin","uusdc:unusd":"usd-coin","uatom:uusd":"cosmos","uatom:unusd":"cosmos","ubnb:uusd":"binancecoin","ubnb:unusd":"binancecoin","uavax:uusd":"avalanche-2","uavax:unusd":"avalanche-2","usol:uusd":"solana","usol:unusd":"solana","uada:uusd":"cardano","uada:unusd":"cardano"}}'
export FEEDER_MNEMONIC="<YOUR_MNEMONIC_HERE>"
export VALIDATOR_ADDRESS="<YOUR_VALOPER_ADDRESS>"
```

Setup service file

```sh
sudo tee /etc/systemd/system/pricefeeder.service<<EOF
[Unit]
Description=Side Protocol Pricefeeder
Requires=network-online.target
After=network-online.target

[Service]
Type=exec
User=admin
ExecStart=/usr/local/bin/pricefeeder
Restart=on-failure
KillSignal=SIGTERM
PermissionsStartOnly=true
LimitNOFILE=65535
Environment=CHAIN_ID='$CHAIN_ID'
Environment=GRPC_ENDPOINT='$GRPC_ENDPOINT'
Environment=WEBSOCKET_ENDPOINT='$WEBSOCKET_ENDPOINT'
Environment=EXCHANGE_SYMBOLS_MAP='$EXCHANGE_SYMBOLS_MAP'
Environment=FEEDER_MNEMONIC='$FEEDER_MNEMONIC'
Environment=VALIDATOR_ADDRESS='$VALIDATOR_ADDRESS'

[Install]
WantedBy=multi-user.target
EOF
```

Start pricefeeder

```sh
sudo systemctl daemon-reload && \
sudo systemctl enable pricefeeder && \
sudo systemctl restart pricefeeder && \
sudo journalctl -u pricefeeder -f -o cat
```

Bind a separate wallet with the following command

```sh
sided tx oracle set-feeder <nibi_address> --from <name_vallet> --fees=5000uside -y
```
