---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/c4e/main_install.sh)
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
git checkout v1.3.1

make install

c4ed config keyring-backend os
c4ed config chain-id perun-1
c4ed init "<Your moniker>" --chain-id perun-1

wget -O $HOME/.c4e-chain/config/genesis.json "https://raw.githubusercontent.com/chain4energy/c4e-chains/main/perun-1/genesis.json"

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
PEERS="084a5c788c9c61541152192d7dfe055c153af642@5.135.141.191:26656,81a3c179ee820d291adebc215d5d1af95b887ec8@65.109.30.185:26656,3c6553a3c45477c2a9902e54069bee7109318b9d@163.172.18.144:26656,68a611fc1d17612e4de6b1232d04568ea3c20a19@77.55.216.80:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
SEEDS="30e98bbcf5bb29ed4e4ff685fa8fa84fa0ddff51@tenderseed.ccvalidators.com:26008"
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
URL="https://snapshots.stake-town.com/c4e/perun-1_latest.tar.lz4"
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
--chain-id=perun-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000uc4e
-y
```
