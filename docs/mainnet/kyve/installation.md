---
cover: ../../.gitbook/assets/kyve-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/kyve/main_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf $HOME/kyve
git clone https://github.com/KYVENetwork/chain.git
cd $HOME/kyve || return
git checkout v1.5.0

make install ENV=mainnet

kyved config keyring-backend os
kyved config chain-id kyve-1
kyved init "Your Moniker" --chain-id kyve-1

# Download genesis and addrbook
curl -Ls https://snapshots-1.stake-town.com/kyve/genesis.json > $HOME/.kyve/config/genesis.json
curl -Ls https://snapshots-1.stake-town.com/kyve/addrbook.json > $HOME/.kyve/config/addrbook.json

APP_TOML="~/.kyve/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.kyve/config/config.toml"
SEEDS="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:11056"
PEERS="24feaba7bf73a2e80d4bec88c1004b9377e28495@65.108.195.213:57656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001ukyve"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.kyve/cosmovisor/genesis/bin
mkdir -p ~/.kyve/cosmovisor/upgrades
cp ~/go/bin/kyved ~/.kyve/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/kyved.service > /dev/null << EOF
[Unit]
Description=Kyve Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=kyved"
Environment="DAEMON_HOME=$HOME/.kyve"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
kyved tendermint unsafe-reset-all --home $HOME/.kyve --keep-addr-book

URL=https://snapshots-1.stake-town.com/kyve/kyve-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.kyve
[[ -f $HOME/.kyve/data/upgrade-info.json ]] && cp $HOME/.kyve/data/upgrade-info.json $HOME/.kyve/cosmovisor/genesis/upgrade-info.json
```

Enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable kyved
sudo systemctl start kyved

sudo journalctl -u kyved -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.kyve/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.kyve/config/config.toml
sudo systemctl restart kyved && sudo journalctl -u kyved -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
kyved keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
kyved keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
kyved tx staking create-validator \
--amount=1000000ukyve \
--pubkey=$(kyved tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=kyve-1 \
--commission-rate=0.1 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.001ukyve \
--gas-adjustment=1.6 \
--gas=auto \
-y
```
