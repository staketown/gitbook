---
cover: ../../.gitbook/assets/ojo banner.jpeg
coverY: 0
---

# Installation

Automatic

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/ojo/install.sh)
```

Manual

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf ojo
git clone https://github.com/ojo-network/ojo.git
cd $HOME/ojo || return
git checkout v0.1.2
make install
ojod version # v0.1.2

ojod config keyring-backend os
ojod config chain-id ojo-devnet
ojod init "<Your moniker>" --chain-id ojo-devnet

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/ojo/genesis.json > $HOME/.ojo/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/ojo/addrbook.json > $HOME/.ojo/config/addrbook.json

APP_TOML="~/.ojo/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $APP_TOML

CONFIG_TOML="~/.ojo/config/config.toml"
SEEDS=""
PEERS="18300f0a5973798c3900fe51ff255bb6bca982f9@ojo-testnet-rpc.stake-town.com:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025uojo"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 30/g' $CONFIG_TOML
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 30/g' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/ojod.service > /dev/null << EOF
[Unit]
Description=Ojo Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which ojod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
URL="https://snapshots-testnet.stake-town.com/ojo/ojo-devnet_latest.tar.lz4"
curl -L $URL | tar -Ilz4 -xf - -C $HOME/.ojo
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.ojo/config/config.toml"
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
sudo systemctl enable ojod
sudo systemctl start ojod

sudo journalctl -u ojod -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.ojo/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.ojo/config/config.toml
sudo systemctl restart ojod && sudo journalctl -u ojod -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
ojod keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
ojod keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
ojod tx staking create-validator \
--amount=1000000uojo \
--pubkey=$(ojod tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=ojo-devnet \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1uojo \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

### Price feeder installation

> ⚠️ save your mnemonic during creation wallet for feeder, you will find it in logs

You must have some deposit on your main wallet just to send some token to created price feeder wallet.

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/ojo/price_feeder_install.sh)
```

### Remove price feeder

> ⚠️ removing price feeder and wallet created for it

```bash
sudo systemctl disable ojo-price-feeder.service && \
sudo systemctl daemon-reload && \
sudo rm /etc/systemd/system/ojo-price-feeder.service && \
sudo rm -rf $HOME/price-feeder && \
sudo rm -rf $HOME/.ojo-price-feeder && \
ojod keys delete price_feeder_wallet
```
