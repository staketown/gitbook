---
cover: ../../.gitbook/assets/composable-banner.jpeg
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
picad keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
picad keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
picad keys list
```

Delete wallet

```bash
picad keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
picad keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
picad keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
picad q bank balances $(picad keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
picad tx staking create-validator \
--amount=1000000000000ppica \
--pubkey=$(picad tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=centauri-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=1000000ppica \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Edit validator

```bash
picad tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=1000000ppica \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Unjail your validator

```bash
picad tx slashing unjail --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Check blocks info processed by your validator

```bash
picad query slashing signing-info $(picad tendermint show-validator)
```

List of active validators

```bash
picad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
picad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
picad q staking validator $(picad keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
picad tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Get rewards and commissions from your validator

```bash
picad tx distribution withdraw-rewards $(picad keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to your validator

```bash
picad tx staking delegate $(picad keys show <YOUR_WALLET_NAME> --bech val -a) 1000000000000ppica --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to validator

```bash
picad tx staking delegate <VALOPER_ADDRESS> 1000000000000ppica --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Redelegate tokens to another validator

```bash
picad tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000000000ppica --from <WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
picad tx staking unbond <VALOPER_ADDRESS> 1000000000000ppica --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Send tokens to another wallet

```bash
picad tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000000000ppica --from <YOUR_WALLET_ADDRESS> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
picad query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
picad tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000000000ppica \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=1000000ppica \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

List of all proposals

```bash
picad query gov proposals
```

Check proposal info by proposal id

```bash
picad query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
picad tx gov deposit 1 1000000000000ppica --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Vote as, **YES**

```bash
picad tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO**

```bash
picad tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
picad tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
picad tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 1000000ppica --gas-adjustment 1.4 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.banksy/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.banksy/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.banksy/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.banksy/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.banksy/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.banksy/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(picad tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.banksy/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.banksy/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
picad status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
picad status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
picad status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
picad tendermint unsafe-reset-all --home $HOME/.banksy --keep-addr-book
```

Delete node

```bash
sudo systemctl stop picad && \
sudo systemctl disable picad && \
sudo rm /etc/systemd/system/picad.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.banksy && \
rm -rf $HOME/composable-centauri
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
picad q staking params
picad q slashing params
```

Check validator private key is correct

```bash
[[ $(picad q staking validator $(picad keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(picad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
picad q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
picad q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable picad
```

Disable service

```bash
sudo systemctl disable picad
```

Start service

```bash
sudo systemctl start picad
```

Stop service

```bash
sudo systemctl stop picad
```

Restart service

```bash
sudo systemctl restart picad
```

Check service status

```bash
sudo systemctl status picad
```

Check service logs

```bash
sudo journalctl -u picad -f -o cat
```
