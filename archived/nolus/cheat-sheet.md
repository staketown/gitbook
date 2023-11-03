---
cover: ../../.gitbook/assets/nolus-defi-banner.png
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
nolusd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
nolusd keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
nolusd keys list
```

Delete wallet

```bash
nolusd keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
nolusd keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
nolusd keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
nolusd q bank balances $(nolusd keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
nolusd tx staking create-validator \
--amount=1000000unls \
--pubkey=$(nolusd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1unls \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

Edit validator

```bash
nolusd tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1unls \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

Unjail your validator

```bash
nolusd tx slashing unjail --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Check blocks info processed by your validator

```bash
nolusd query slashing signing-info $(nolusd tendermint show-validator)
```

List of active validators

```bash
nolusd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
nolusd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
nolusd q staking validator $(nolusd keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
nolusd tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Get rewards and commissions from your validator

```bash
nolusd tx distribution withdraw-rewards $(nolusd keys show <WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Delegate tokens to your validator

```bash
nolusd tx staking delegate $(nolusd keys show <YOUR_WALLET_NAME> --bech val -a) 1000000unls --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Delegate tokens to validator

```bash
nolusd tx staking delegate <VALOPER_ADDRESS> 1000000unls --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Redelegate tokens to another validator

```bash
nolusd tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000unls --from <WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
nolusd tx staking unbond <VALOPER_ADDRESS> 1000000unls --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Send tokens to another wallet

```bash
nolusd tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000unls --from <YOUR_WALLET_ADDRESS> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
nolusd query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
nolusd tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000unls \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0.1unls \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

List of all proposals

```bash
nolusd query gov proposals
```

Check proposal info by proposal id

```bash
nolusd query gov proposal <proposal_id>
```

Vote as, **YES**

```bash
nolusd tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Vote as, **NO**

```bash
nolusd tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
nolusd tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
nolusd tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0.1unls --gas-adjustment 1.5 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.nolus/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.nolus/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.nolus/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.nolus/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.nolus/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.nolus/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(nolusd tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.nolus/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.nolus/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
nolusd status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
nolusd status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
nolusd status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book
```

Delete node

```bash
sudo systemctl stop nolusd && \
sudo systemctl disable nolusd && \
sudo rm /etc/systemd/system/nolusd.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.nolus && \
rm -rf $HOME/nolus
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
nolusd q staking params
nolusd q slashing params
```

Search all output transactions by address

```bash
nolusd q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
nolusd q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable nolusd
```

Disable service

```bash
sudo systemctl disable nolusd
```

Start service

```bash
sudo systemctl start nolusd
```

Stop service

```bash
sudo systemctl stop nolusd
```

Restart service

```bash
sudo systemctl restart nolusd
```

Check service status

```bash
sudo systemctl status nolusd
```

Check service logs

```bash
sudo journalctl -u nolusd -f -o cat
```
