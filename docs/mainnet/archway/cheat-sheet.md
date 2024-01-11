---
cover: ../../.gitbook/assets/archway-banner.jpeg
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
archwayd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
archwayd keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
archwayd keys list
```

Delete wallet

```bash
archwayd keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
archwayd keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
archwayd keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
archwayd q bank balances $(archwayd keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
archwayd tx staking create-validator \
--amount=1000000000000000000aarch \
--pubkey=$(archwayd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=archway-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=1000000000000aarch \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Edit validator

```bash
archwayd tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=1000000000000aarch \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Unjail your validator

```bash
archwayd tx slashing unjail --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Check blocks info processed by your validator

```bash
archwayd query slashing signing-info $(archwayd tendermint show-validator)
```

List of active validators

```bash
archwayd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
archwayd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
archwayd q staking validator $(archwayd keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
archwayd tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Get rewards and commissions from your validator

```bash
archwayd tx distribution withdraw-rewards $(archwayd keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to your validator

```bash
archwayd tx staking delegate $(archwayd keys show <YOUR_WALLET_NAME> --bech val -a) 1000000000000000000aarch --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to validator

```bash
archwayd tx staking delegate <VALOPER_ADDRESS> 1000000000000000000aarch --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Redelegate tokens to another validator

```bash
archwayd tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000000000000000aarch --from <WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
archwayd tx staking unbond <VALOPER_ADDRESS> 1000000000000000000aarch --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Send tokens to another wallet

```bash
archwayd tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000000000000000aarch --from <YOUR_WALLET_ADDRESS> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
archwayd query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
archwayd tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000000000000000aarch \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=1000000000000aarch \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

List of all proposals

```bash
archwayd query gov proposals
```

Check proposal info by proposal id

```bash
archwayd query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
archwayd tx gov deposit 1 1000000000000000000aarch --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Vote as, **YES**

```bash
archwayd tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO**

```bash
archwayd tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
archwayd tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
archwayd tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 1000000000000aarch --gas-adjustment 1.4 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.archway/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.archway/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.archway/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.archway/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.archway/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.archway/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(archwayd tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.archway/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.archway/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
archwayd status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
archwayd status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
archwayd status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
archwayd tendermint unsafe-reset-all --home $HOME/.archway --keep-addr-book
```

Delete node

```bash
sudo systemctl stop archwayd && \
sudo systemctl disable archwayd && \
sudo rm /etc/systemd/system/archwayd.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.archway && \
rm -rf $HOME/archway
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
archwayd q staking params
archwayd q slashing params
```

Check validator private key is correct

```bash
[[ $(archwayd q staking validator $(archwayd keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(archwayd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
archwayd q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
archwayd q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable archwayd
```

Disable service

```bash
sudo systemctl disable archwayd
```

Start service

```bash
sudo systemctl start archwayd
```

Stop service

```bash
sudo systemctl stop archwayd
```

Restart service

```bash
sudo systemctl restart archwayd
```

Check service status

```bash
sudo systemctl status archwayd
```

Check service logs

```bash
sudo journalctl -u archwayd -f -o cat
```
