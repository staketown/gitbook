---
cover: ../../.gitbook/assets/crossfi-banner.png
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/crossfi/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
wget https://github.com/crossfichain/crossfi-node/releases/download/v0.3.0-prebuild9/crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz
mkdir $HOME/crossfi_tmp
tar -xvf crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz -C $HOME/crossfi_tmp
mv $HOME/crossfi_tmp/bin/crossfid ~/go/bin/crossfid

rm crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz
rm -rf $HOME/crossfi_tmp

crossfid config keyring-backend os
crossfid config chain-id crossfi-evm-testnet-1
crossfid init "Your Moniker" --chain-id crossfi-evm-testnet-1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/crossfi/genesis.json > $HOME/.mineplex-chain/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/crossfi/addrbook.json > $HOME/.mineplex-chain/config/addrbook.json

APP_TOML="~/.mineplex-chain/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = 19|g' $APP_TOML

CONFIG_TOML="~/.mineplex-chain/config/config.toml"
SEEDS=""
PEERS="634780077115040a5946d8d22e98910fb68205a2@65.109.65.248:52656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "5000000000mpx"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

# Install cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.mineplex-chain/cosmovisor/genesis/bin
mkdir -p ~/.mineplex-chain/cosmovisor/upgrades
cp ~/go/bin/crossfid ~/.mineplex-chain/cosmovisor/genesis/bin

sudo tee /etc/systemd/system/crossfid.service > /dev/null << EOF
[Unit]
Description=Cross Finance Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=crossfid"
Environment="DAEMON_HOME=$HOME/.mineplex-chain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
crossfid tendermint unsafe-reset-all --home $HOME/.mineplex-chain --keep-addr-book

URL=https://snapshots-testnet.stake-town.com/crossfi/crossfi-evm-testnet-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.mineplex-chain
[[ -f $HOME/.mineplex-chain/data/upgrade-info.json ]] && cp $HOME/.mineplex-chain/data/upgrade-info.json $HOME/.mineplex-chain/cosmovisor/genesis/upgrade-info.json
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.mineplex-chain/config/config.toml"
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
sudo systemctl enable crossfid
sudo systemctl start crossfid

sudo journalctl -u crossfid -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.mineplex-chain/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.mineplex-chain/config/config.toml
sudo systemctl restart crossfid && sudo journalctl -u crossfid -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
crossfid keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
crossfid keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
crossfid tx staking create-validator \
--amount=1000000000000000000mpx \
--pubkey=$(crossfid tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=crossfi-evm-testnet-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=5000000000000mpx \
--gas-adjustment=1.4 \
--gas=auto \
-y
```
