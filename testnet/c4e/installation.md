---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/c4e/test_install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd $HOME || return
rm -rf c4e-chain
git clone https://github.com/chain4energy/c4e-chain
cd $HOME/c4e-chain || return
git checkout v1.3.0
make install
c4ed version # v1.3.0

c4ed config keyring-backend os
c4ed config chain-id babajaga-1
c4ed init "<Your moniker>" --chain-id babajaga-1

curl -s https://snapshots-testnet.stake-town.com/c4e/genesis.json > $HOME/.c4e-chain/config/genesis.json
curl -s https://snapshots-testnet.stake-town.com/c4e/addrbook.json > $HOME/.c4e-chain/config/addrbook.json

APP_TOML=$HOME/.c4e-chain/config/app.toml
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "19"|g' $APP_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0025uc4e"|g' $APP_TOML

CONFIG_TOML=$HOME/.c4e-chain/config/config.toml
PEERS="de18fc6b4a5a76bd30f65ebb28f880095b5dd58b@66.70.177.76:36656,33f90a0ac7e8f48305ea7e64610b789bbbb33224@151.80.19.186:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
SEEDS=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/c4ed.service > /dev/null << EOF
[Unit]
Description=C4E Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which c4ed) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book

# Add snapshot here
URL="https://snapshots-testnet.stake-town.com/c4e/babajaga-1_latest.tar.lz4"
curl $URL | lz4 -dc - | tar -xf - -C $HOME/.c4e-chain
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.c4e-chain/config/config.toml"
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
sudo systemctl enable c4ed
sudo systemctl start c4ed

sudo journalctl -u c4ed -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.c4e-chain/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.c4e-chain/config/config.toml
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

## Wallet creation

Create wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
c4ed keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
c4ed keys add <YOUR_WALLET_NAME> --recover
```

## Validator creation

After successful synchronisation we can proceed with validation creation.

```bash
c4ed tx staking create-validator \
--amount=1000000uc4e \
--pubkey=$(c4ed tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=babajaga-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000uc4e
-y
```
