---
cover: ../../.gitbook/assets/quasar-banner.jpeg
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
quasarnoded keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
quasarnoded keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
quasarnoded keys list
```

Delete wallet

```bash
quasarnoded keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
quasarnoded keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
quasarnoded keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
quasarnoded q bank balances $(quasarnoded keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
quasarnoded tx staking create-validator \
--amount=1000000uqsr \
--pubkey=$(quasarnoded tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=quasar-test-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0uqsr \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Edit validator

```bash
quasarnoded tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=0uqsr \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Unjail your validator

```bash
quasarnoded tx slashing unjail --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Check blocks info processed by your validator

```bash
quasarnoded query slashing signing-info $(quasarnoded tendermint show-validator)
```

List of active validators

```bash
quasarnoded q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
quasarnoded q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
quasarnoded q staking validator $(quasarnoded keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
quasarnoded tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Get rewards and commissions from your validator

```bash
quasarnoded tx distribution withdraw-rewards $(quasarnoded keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to your validator

```bash
quasarnoded tx staking delegate $(quasarnoded keys show <YOUR_WALLET_NAME> --bech val -a) 1000000uqsr --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to validator

```bash
quasarnoded tx staking delegate <VALOPER_ADDRESS> 1000000uqsr --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Redelegate tokens to another validator

```bash
quasarnoded tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000uqsr --from <WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
quasarnoded tx staking unbond <VALOPER_ADDRESS> 1000000uqsr --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Send tokens to another wallet

```bash
quasarnoded tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000uqsr --from <YOUR_WALLET_ADDRESS> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
quasarnoded query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
quasarnoded tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000uqsr \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0uqsr \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

List of all proposals

```bash
quasarnoded query gov proposals
```

Check proposal info by proposal id

```bash
quasarnoded query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
quasarnoded tx gov deposit 1 1000000uqsr --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Vote as, **YES**

```bash
quasarnoded tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO**

```bash
quasarnoded tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
quasarnoded tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
quasarnoded tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0uqsr --gas-adjustment 1.4 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.quasarnode/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.quasarnode/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.quasarnode/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.quasarnode/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.quasarnode/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.quasarnode/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(quasarnoded tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.quasarnode/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.quasarnode/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
quasarnoded status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
quasarnoded status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
quasarnoded status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
quasarnoded tendermint unsafe-reset-all --home $HOME/.quasarnode --keep-addr-book
```

Delete node

```bash
sudo systemctl stop quasarnoded && \
sudo systemctl disable quasarnoded && \
sudo rm /etc/systemd/system/quasarnoded.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.quasarnode && \
rm -rf $HOME/quasar
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
quasarnoded q staking params
quasarnoded q slashing params
```

Check validator private key is correct

```bash
[[ $(quasarnoded q staking validator $(quasarnoded keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(quasarnoded status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
quasarnoded q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
quasarnoded q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable quasarnoded
```

Disable service

```bash
sudo systemctl disable quasarnoded
```

Start service

```bash
sudo systemctl start quasarnoded
```

Stop service

```bash
sudo systemctl stop quasarnoded
```

Restart service

```bash
sudo systemctl restart quasarnoded
```

Check service status

```bash
sudo systemctl status quasarnoded
```

Check service logs

```bash
sudo journalctl -u quasarnoded -f -o cat
```
