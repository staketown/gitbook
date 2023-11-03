---
cover: ../../.gitbook/assets/defund-banner.jpeg
coverY: 0
---

# Installation

Automatic

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/defund/install.sh)
```

Manual installation

```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/staketown/cosmos/master/utils/go_install.sh")
source .bash_profile

cd || return
rm -rf defund
git clone https://github.com/defund-labs/defund.git
cd defund || return
git checkout # 0.2.6
make install
defundd version # 0.2.6

defundd config keyring-backend os
defundd config chain-id orbit-alpha-1
defundd init "<Your moniker>" --chain-id orbit-alpha-1

# Download genesis and addrbook
curl -Ls https://snapshots-testnet.stake-town.com/defund/genesis.json > $HOME/.defund/config/genesis.json
curl -Ls https://snapshots-testnet.stake-town.com/defund/addrbook.json > $HOME/.defund/config/addrbook.json

APP_TOML="~/.defund/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|g' $APP_TOML
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $APP_TOML

CONFIG_TOML="~/.defund/config/config.toml"
SEEDS="f902d7562b7687000334369c491654e176afd26d@170.187.157.19:26656,2b76e96658f5e5a5130bc96d63f016073579b72d@rpc-1.defund.nodes.guru:45656"
PEERS=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025ufetf"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 30/g' $CONFIG_TOML
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 30/g' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML

sudo tee /etc/systemd/system/defundd.service > /dev/null << EOF
[Unit]
Description=Defund Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

# Snapshots
URL="https://snapshots-testnet.stake-town.com/defund/orbit-alpha-1_latest.tar.lz4"
curl -L $URL | tar -Ilz4 -xf - -C $HOME/.defund
```

**(Optional) Configure timeouts for processing blocks**

```bash
CONFIG_TOML="~/.defund/config/config.toml"
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
sudo systemctl enable defundd
sudo systemctl start defundd

sudo journalctl -u defundd -f -o cat
```

> After successful synchronisation we recommend to turn off **snapshot\_interval** and state sync, this will save space on your hardware.

```bash
snapshot_interval=0
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.defund/config/app.toml
sed -i 's|^enable *=.*|enable = false|' $HOME/.defund/config/config.toml
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```

### Wallet creation

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
defundd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
defundd keys add <YOUR_WALLET_NAME> --recover
```

### Validator creation

After successful synchronisation we can proceed with validation creation.

Create validator

```bash
defundd tx staking create-validator \
--amount=50ufetf \
--pubkey=$(defundd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=orbit-alpha-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=20ufetf
-y
```
