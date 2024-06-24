---
cover: ../../.gitbook/assets/umee-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/umee/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf umee
git clone https://github.com/umee-network/umee.git
cd umee || return
git checkout v6.5.0

make install

umeed config keyring-backend os
umeed config chain-id umee-1
umeed init "Your Moniker" --chain-id umee-1

# Download genesis and addrbook
curl -Ls https://snapshots.stake-town.com/umee/genesis.json > $HOME/.umee/config/genesis.json
curl -Ls https://snapshots.stake-town.com/umee/addrbook.json > $HOME/.umee/config/addrbook.json

APP_TOML="~/.umee/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.umee/config/config.toml"
SEEDS="400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@umee.rpc.kjnodes.com:16259"
PEERS="635debe6c5cbcb6861b6c8b32c47d8ee84d99c16@138.201.21.197:29656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.1uumee"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.umee/cosmovisor/genesis/bin
mkdir -p ~/.umee/cosmovisor/upgrades
cp ~/go/bin/umeed ~/.umee/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/umeed.service > /dev/null << EOF
[Unit]
Description=Umee Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=umeed"
Environment="DAEMON_HOME=$HOME/.umee"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
umeed tendermint unsafe-reset-all --home $HOME/.umee --keep-addr-book

URL=https://snapshots.stake-town.com/umee/umee-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.umee
[[ -f $HOME/.umee/data/upgrade-info.json ]] && cp $HOME/.umee/data/upgrade-info.json $HOME/.umee/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.umee/config/config.toml"
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
sudo systemctl enable umeed
sudo systemctl start umeed

sudo journalctl -u umeed -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.umee/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.umee/config/config.toml
sudo systemctl restart umeed && sudo journalctl -u umeed -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
umeed keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
umeed keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
umeed tx staking create-validator \
--amount=1000000uumee \
--pubkey=$(umeed tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=umee-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1uumee \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

### Price feeder installation

> ⚠️ save your mnemonic during creation wallet for feeder, you will find it in logs

You must have some deposit on your main wallet just to send some token to created price feeder wallet.

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/umee/price_feeder_install.sh)
```

### Remove price feeder

> ⚠️ removing price feeder and wallet created for it

```bash
sudo systemctl disable umee-price-feeder.service && \
sudo systemctl daemon-reload && \
sudo rm /etc/systemd/system/umee-price-feeder.service && \
sudo rm -rf $HOME/price-feeder && \
sudo rm -rf $HOME/.umee-price-feeder && \
umeed keys delete price_feeder_wallet
```
