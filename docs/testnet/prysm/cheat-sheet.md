---
cover: ../../.gitbook/assets/prysm-banner.png
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
prysmd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
prysmd keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
prysmd keys list
```

Delete wallet

```bash
prysmd keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
prysmd keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
prysmd keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
prysmd q bank balances $(prysmd keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
prysmd tx staking create-validator \
--amount=1000000uprysm \
--pubkey=$(prysmd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=prysm-devnet-1 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.0002uprysm \
--gas-adjustment=1.6 \
--gas=auto \
-y
```

Edit validator

```bash
prysmd tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=0.0002uprysm \
--gas-adjustment=1.6 \
--gas=auto \
-y
```

Unjail your validator

```bash
prysmd tx slashing unjail --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Check blocks info processed by your validator

```bash
prysmd query slashing signing-info $(prysmd tendermint show-validator)
```

List of active validators

```bash
prysmd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
prysmd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
prysmd q staking validator $(prysmd keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
prysmd tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Get rewards and commissions from your validator

```bash
prysmd tx distribution withdraw-rewards $(prysmd keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Delegate tokens to your validator

```bash
prysmd tx staking delegate $(prysmd keys show <YOUR_WALLET_NAME> --bech val -a) 1000000uprysm --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Delegate tokens to validator

```bash
prysmd tx staking delegate <VALOPER_ADDRESS> 1000000uprysm --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Redelegate tokens to another validator

```bash
prysmd tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000uprysm --from <WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
prysmd tx staking unbond <VALOPER_ADDRESS> 1000000uprysm --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Send tokens to another wallet

```bash
prysmd tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000uprysm --from <YOUR_WALLET_ADDRESS> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
prysmd query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
prysmd tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000uprysm \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0.0002uprysm \
--gas-adjustment=1.6 \
--gas=auto \
-y
```

List of all proposals

```bash
prysmd query gov proposals
```

Check proposal info by proposal id

```bash
prysmd query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
prysmd tx gov deposit 1 1000000uprysm --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Vote as, **YES**

```bash
prysmd tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Vote as, **NO**

```bash
prysmd tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
prysmd tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
prysmd tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0.0002uprysm --gas-adjustment 1.6 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.prysm/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.prysm/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.prysm/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.prysm/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.prysm/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.prysm/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(prysmd tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.prysm/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.prysm/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
prysmd status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
prysmd status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
prysmd status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
prysmd tendermint unsafe-reset-all --home $HOME/.prysm --keep-addr-book
```

Delete node

```bash
sudo systemctl stop prysmd && \
sudo systemctl disable prysmd && \
sudo rm /etc/systemd/system/prysmd.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.prysm && \
rm -rf $HOME/prysm
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
prysmd q staking params
prysmd q slashing params
```

Check validator private key is correct

```bash
[[ $(prysmd q staking validator $(prysmd keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(prysmd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
prysmd q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
prysmd q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable prysmd
```

Disable service

```bash
sudo systemctl disable prysmd
```

Start service

```bash
sudo systemctl start prysmd
```

Stop service

```bash
sudo systemctl stop prysmd
```

Restart service

```bash
sudo systemctl restart prysmd
```

Check service status

```bash
sudo systemctl status prysmd
```

Check service logs

```bash
sudo journalctl -u prysmd -f -o cat
```
