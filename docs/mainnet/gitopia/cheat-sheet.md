---
cover: ../../.gitbook/assets/gitopia-banner.png
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
gitopiad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
gitopiad keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
gitopiad keys list
```

Delete wallet

```bash
gitopiad keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
gitopiad keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
gitopiad keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
gitopiad q bank balances $(gitopiad keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
gitopiad tx staking create-validator \
--amount=1000000ulore \
--pubkey=$(gitopiad tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1ulore \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

Edit validator

```bash
gitopiad tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=0.1ulore \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

Unjail your validator

```bash
gitopiad tx slashing unjail --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Check blocks info processed by your validator

```bash
gitopiad query slashing signing-info $(gitopiad tendermint show-validator)
```

List of active validators

```bash
gitopiad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
gitopiad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
gitopiad q staking validator $(gitopiad keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
gitopiad tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Get rewards and commissions from your validator

```bash
gitopiad tx distribution withdraw-rewards $(gitopiad keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Delegate tokens to your validator

```bash
gitopiad tx staking delegate $(gitopiad keys show <YOUR_WALLET_NAME> --bech val -a) 1000000ulore --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Delegate tokens to validator

```bash
gitopiad tx staking delegate <VALOPER_ADDRESS> 1000000ulore --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Redelegate tokens to another validator

```bash
gitopiad tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000ulore --from <WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
gitopiad tx staking unbond <VALOPER_ADDRESS> 1000000ulore --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Send tokens to another wallet

```bash
gitopiad tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000ulore --from <YOUR_WALLET_ADDRESS> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
gitopiad query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
gitopiad tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000ulore \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0.1ulore \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

List of all proposals

```bash
gitopiad query gov proposals
```

Check proposal info by proposal id

```bash
gitopiad query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
gitopiad tx gov deposit 1 1000000ulore --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y 
```

Vote as, **YES**

```bash
gitopiad tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Vote as, **NO**

```bash
gitopiad tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
gitopiad tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
gitopiad tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0.1ulore --gas-adjustment 1.5 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.gitopia/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.gitopia/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.gitopia/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.gitopia/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.gitopia/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.gitopia/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(gitopiad tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.gitopia/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.gitopia/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
gitopiad status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
gitopiad status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
gitopiad status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia --keep-addr-book
```

Delete node

```bash
sudo systemctl stop gitopiad && \
sudo systemctl disable gitopiad && \
sudo rm /etc/systemd/system/gitopiad.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.gitopia && \
rm -rf $HOME/gitopia
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
gitopiad q staking params
gitopiad q slashing params
```

```bash
```

Check validator private key is correct

```bash
[[ $(gitopiad q staking validator $(gitopiad keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(gitopiad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
gitopiad q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
gitopiad q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable gitopiad
```

Disable service

```bash
sudo systemctl disable gitopiad
```

Start service

```bash
sudo systemctl start gitopiad
```

Stop service

```bash
sudo systemctl stop gitopiad
```

Restart service

```bash
sudo systemctl restart gitopiad
```

Check service status

```bash
sudo systemctl status gitopiad
```

Check service logs

```bash
sudo journalctl -u gitopiad -f -o cat
```
