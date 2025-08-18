---
cover: ../../.gitbook/assets/celestia-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/celestia/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd $HOME/celestia-app || return
git checkout v5.0.2

make install

celestia-appd config keyring-backend os
celestia-appd config chain-id celestia
celestia-appd init "Your Moniker" --chain-id celestia

# Download genesis and addrbook
curl -Ls https://snapshots.stake-town.com/celestia/genesis.json > $HOME/.celestia-app/config/genesis.json
curl -Ls https://snapshots.stake-town.com/celestia/addrbook.json > $HOME/.celestia-app/config/addrbook.json

APP_TOML="~/.celestia-app/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "nothing"|g' $APP_TOML
sed -i -e "s|^target_height_duration *=.*|timeout_commit = \"11s\"|" $HOME/.celestia-app/config/config.toml

CONFIG_TOML="~/.celestia-app/config/config.toml"
SEEDS=""
PEERS="761973c2e641b6770c727ab800b3fb2124b29328@144.76.15.125:34656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.002utia"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.celestia-app/cosmovisor/genesis/bin
mkdir -p ~/.celestia-app/cosmovisor/upgrades
cp ~/go/bin/celestia-appd ~/.celestia-app/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/celestia-appd.service > /dev/null << EOF
[Unit]
Description=Celestia Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --v2-upgrade-height 2371495
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=celestia-appd"
Environment="DAEMON_HOME=$HOME/.celestia-app"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app --keep-addr-book

URL=https://snapshots.stake-town.com/celestia/celestia_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.celestia-app
[[ -f $HOME/.celestia-app/data/upgrade-info.json ]] && cp $HOME/.celestia-app/data/upgrade-info.json $HOME/.celestia-app/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.celestia-app/config/config.toml"
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
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd

sudo journalctl -u celestia-appd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.celestia-app/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.celestia-app/config/config.toml
sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
celestia-appd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
celestia-appd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
celestia-appd tx staking create-validator \
--amount=1000000utia \
--pubkey=$(celestia-appd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=celestia \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.002utia \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

# Install Bridge Node (Shwap)

Download and build binaries
```bash
cd $HOME
rm -rf celestia-node
git clone https://github.com/celestiaorg/celestia-node.git
cd celestia-node
git checkout v0.24.1
make build
sudo mv build/celestia $HOME/go/bin
make cel-key
sudo mv cel-key $HOME/go/bin
```

Add new bridge wallet
```bash
cel-key add bridge-wallet --node.type bridge --p2p.network celestia
```

Recover existent bridge wallet (in case you already have bridge wallet)
```bash
cel-key add bridge-wallet --node.type bridge --p2p.network celestia --recover
```

> ⚠️ Once you start the Bridge Node, a wallet key will be generated. You need to fund that address with testnet tokens (could be used faucet) to pay for PayForBlob transactions

Initialize Bridge node
```bash
celestia bridge init \
--keyring.keyname bridge-wallet \
--core.ip http://localhost \
--core.port 9090 \
--p2p.network celestia \
--rpc.port 26658 \
--gateway.port 26659
```

Create service
```bash
sudo tee /etc/systemd/system/celestia-bridge.service > /dev/null << EOF
[Unit]
Description=Celestia Bridge Node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which celestia) bridge start \\
--keyring.keyname bridge-wallet \\
--core.ip http://localhost \\
--core.port 9090 \\
--p2p.network celestia \\
--rpc.port 26658 \\
--gateway.port 26659 \\
--metrics.tls=true \\
--metrics \\
--metrics.endpoint otel.celestia.observer \\
--archival
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Load bridge node service file
```bash
sudo systemctl daemon-reload
sudo systemctl enable celestia-bridge.service
```

Start Bridge node
```bash
systemctl restart celestia-bridge.service
```

Check bridge node logs
```bash
journalctl -fu celestia-bridge.service -o cat
```