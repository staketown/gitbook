---
cover: ../../.gitbook/assets/arkeo-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/arkeo/install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

wget https://snapshots-testnet.stake-town.com/arkeo/arkeod
chmod +x arkeod
mv arkeod $HOME/go/bin/

arkeod version # 1

arkeod config keyring-backend os
arkeod config chain-id arkeo
arkeod init "<Your moniker>" --chain-id arkeo

curl -s http://seed.arkeo.network:26657/genesis | jq '.result.genesis' > $HOME/.arkeo/config/genesis.json
curl -s https://snapshots-testnet.stake-town.com/arkeo/addrbook.json > $HOME/.arkeo/config/addrbook.json

APP_TOML=$HOME/.arkeo/config/app.toml
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "19"|g' $APP_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001uarkeo"|g' $APP_TOML

CONFIG_TOML=$HOME/.arkeo/config/config.toml
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
SEEDS="20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22856"
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which arkeod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

arkeod tendermint unsafe-reset-all --home $HOME/.arkeo --keep-addr-book

# Add snapshot here
URL="https://snapshots-testnet.stake-town.com/arkeo/arkeo_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.arkeo
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.arkeo/config/config.toml"
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
sudo systemctl enable arkeod
sudo systemctl start arkeod

sudo journalctl -u arkeod -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.arkeo/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.arkeo/config/config.toml
systemctl restart arkeod && journalctl -u arkeod -f -o cat
```

## Wallet creation

Create wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
arkeod keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
arkeod keys add <YOUR_WALLET_NAME> --recover
```

## Validator creation

After successful synchronisation we can proceed with validation creation.

```bash
arkeod tx staking create-validator \
--amount=1000000uarkeo \
--pubkey=$(arkeod tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=arkeo \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1uarkeo \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
