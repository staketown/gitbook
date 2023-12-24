---
cover: ../../.gitbook/assets/quicksilver-banner.png
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
quicksilverd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
quicksilverd keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
quicksilverd keys list
```

Delete wallet

```bash
quicksilverd keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
quicksilverd keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
quicksilverd keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
quicksilverd q bank balances $(quicksilverd keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
quicksilverd tx staking create-validator \
--amount=1000000uqck \
--pubkey=$(quicksilverd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=rhye-2 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.0001uqck \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Edit validator

```bash
quicksilverd tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=0.0001uqck \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Unjail your validator

```bash
quicksilverd tx slashing unjail --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Check blocks info processed by your validator

```bash
quicksilverd query slashing signing-info $(quicksilverd tendermint show-validator)
```

List of active validators

```bash
quicksilverd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
quicksilverd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
quicksilverd q staking validator $(quicksilverd keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
quicksilverd tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Get rewards and commissions from your validator

```bash
quicksilverd tx distribution withdraw-rewards $(quicksilverd keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to your validator

```bash
quicksilverd tx staking delegate $(quicksilverd keys show <YOUR_WALLET_NAME> --bech val -a) 1000000uqck --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to validator

```bash
quicksilverd tx staking delegate <VALOPER_ADDRESS> 1000000uqck --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Redelegate tokens to another validator

```bash
quicksilverd tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000uqck --from <WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
quicksilverd tx staking unbond <VALOPER_ADDRESS> 1000000uqck --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Send tokens to another wallet

```bash
quicksilverd tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000uqck --from <YOUR_WALLET_ADDRESS> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
quicksilverd query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
quicksilverd tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000uqck \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0.0001uqck \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

List of all proposals

```bash
quicksilverd query gov proposals
```

Check proposal info by proposal id

```bash
quicksilverd query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
quicksilverd tx gov deposit 1 1000000uqck --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Vote as, **YES**

```bash
quicksilverd tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO**

```bash
quicksilverd tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
quicksilverd tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
quicksilverd tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0.0001uqck --gas-adjustment 1.4 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.quicksilverd/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.quicksilverd/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.quicksilverd/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.quicksilverd/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.quicksilverd/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.quicksilverd/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(quicksilverd tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.quicksilverd/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.quicksilverd/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
quicksilverd status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
quicksilverd status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
quicksilverd status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
quicksilverd tendermint unsafe-reset-all --home $HOME/.quicksilverd --keep-addr-book
```

Delete node

```bash
sudo systemctl stop quicksilverd && \
sudo systemctl disable quicksilverd && \
sudo rm /etc/systemd/system/quicksilverd.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.quicksilverd && \
rm -rf $HOME/quicksilver
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
quicksilverd q staking params
quicksilverd q slashing params
```

Check validator private key is correct

```bash
[[ $(quicksilverd q staking validator $(quicksilverd keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(quicksilverd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
quicksilverd q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
quicksilverd q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable quicksilverd
```

Disable service

```bash
sudo systemctl disable quicksilverd
```

Start service

```bash
sudo systemctl start quicksilverd
```

Stop service

```bash
sudo systemctl stop quicksilverd
```

Restart service

```bash
sudo systemctl restart quicksilverd
```

Check service status

```bash
sudo systemctl status quicksilverd
```

Check service logs

```bash
sudo journalctl -u quicksilverd -f -o cat
```
