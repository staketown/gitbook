---
cover: ../../.gitbook/assets/babylon-banner.jpeg
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
babylond keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
babylond keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
babylond keys list
```

Delete wallet

```bash
babylond keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
babylond keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
babylond keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
babylond q bank balances $(babylond keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
babylond tx staking create-validator \
--amount=1000000uabbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=bbn-test-5 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.025ubbn \
--gas-adjustment=1.6 \
--gas=auto \
-y
```

Edit validator

```bash
babylond tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=0.025ubbn \
--gas-adjustment=1.6 \
--gas=auto \
-y
```

Unjail your validator

```bash
babylond tx slashing unjail --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Check blocks info processed by your validator

```bash
babylond query slashing signing-info $(babylond tendermint show-validator)
```

List of active validators

```bash
babylond q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
babylond q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
babylond q staking validator $(babylond keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
babylond tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Get rewards and commissions from your validator

```bash
babylond tx distribution withdraw-rewards $(babylond keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Delegate tokens to your validator

```bash
babylond tx staking delegate $(babylond keys show <YOUR_WALLET_NAME> --bech val -a) 1000000uabbn --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Delegate tokens to validator

```bash
babylond tx staking delegate <VALOPER_ADDRESS> 1000000uabbn --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Redelegate tokens to another validator

```bash
babylond tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000uabbn --from <WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
babylond tx staking unbond <VALOPER_ADDRESS> 1000000uabbn --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Send tokens to another wallet

```bash
babylond tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000uabbn --from <YOUR_WALLET_ADDRESS> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
babylond query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
babylond tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000uabbn \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0.025ubbn \
--gas-adjustment=1.6 \
--gas=auto \
-y
```

List of all proposals

```bash
babylond query gov proposals
```

Check proposal info by proposal id

```bash
babylond query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
babylond tx gov deposit 1 1000000uabbn --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Vote as, **YES**

```bash
babylond tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Vote as, **NO**

```bash
babylond tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
babylond tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
babylond tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0.025ubbn --gas-adjustment 1.6 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.babylond/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.babylond/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.babylond/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.babylond/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.babylond/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.babylond/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(babylond tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.babylond/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.babylond/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
babylond status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
babylond status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
babylond status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
babylond tendermint unsafe-reset-all --home $HOME/.babylond --keep-addr-book
```

Delete node

```bash
sudo systemctl stop babylond && \
sudo systemctl disable babylond && \
sudo rm /etc/systemd/system/babylond.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.babylond && \
rm -rf $HOME/babylon
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
babylond q staking params
babylond q slashing params
```

Check validator private key is correct

```bash
[[ $(babylond q staking validator $(babylond keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(babylond status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
babylond q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
babylond q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable babylond
```

Disable service

```bash
sudo systemctl disable babylond
```

Start service

```bash
sudo systemctl start babylond
```

Stop service

```bash
sudo systemctl stop babylond
```

Restart service

```bash
sudo systemctl restart babylond
```

Check service status

```bash
sudo systemctl status babylond
```

Check service logs

```bash
sudo journalctl -u babylond -f -o cat
```
