---
cover: ../../.gitbook/assets/nibiru-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/nibiru/install.sh)
```

Manual installation

Setting up the environment and nodes

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staktown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd || return
rm -rf nibiru
git clone <https://github.com/NibiruChain/nibiru>
cd nibiru || return
git checkout v0.19.2
make install
nibid version # v0.19.2

nibid config keyring-backend os
nibid config chain-id nibiru-itn-1
nibid init "<Your moniker>" --chain-id nibiru-itn-1

curl -s https://snapshots-testnet.stake-town.com/nibiru/genesis.json> > $HOME/.nibid/config/genesis.json
curl -s https://snapshots-testnet.stake-town.com/nibiru/addrbook.json> > $HOME/.nibid/config/addrbook.json

APP_TOML="~/.nibid/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 1000|g' $APP_TOML

CONFIG_TOML="~/.nibid/config/config.toml"
SEEDS="a431d3d1b451629a21799963d9eb10d83e261d2c@seed-1.itn-1.nibiru.fi:26656,6a78a2a5f19c93661a493ecbe69afc72b5c54117@seed-2.itn-1.nibiru.fi:26656"
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025unibi"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 30/g' $CONFIG_TOML
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 30/g' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/nibid.service > /dev/null << EOF
[Unit]
Description=Quasar Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book

# Add snapshot here
URL="https://snapshots-testnet.stake-town.com/nibiru/nibiru-itn-1_latest.tar.lz4"
curl $URL | lz4 -dc - | tar -xf - -C $HOME/.nibid
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.nibid/config/config.toml"
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
sudo systemctl enable nibid
sudo systemctl start nibid

sudo journalctl -u nibid -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
nibid keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
nibid keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

```bash
nibid tx staking create-validator \
--amount=1000000unibi \
--pubkey=$(nibid tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=nibiru-itn-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1unibi \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
