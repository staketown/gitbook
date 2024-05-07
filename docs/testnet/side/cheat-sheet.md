---
cover: ../../.gitbook/assets/side-banner.png
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
sided keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
sided keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
sided keys list
```

Delete wallet

```bash
sided keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
sided keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
sided keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
sided q bank balances $(sided keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
sided tx staking create-validator \
--amount=1000000uside \
--pubkey=$(sided tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=S2-testnet-2 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000uside \
-y
```

Edit validator

```bash
sided tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--fees=5000uside \
-y
```

Unjail your validator

```bash
sided tx slashing unjail --from <YOUR_WALLET> --fees=5000uside -y
```

Check blocks info processed by your validator

```bash
sided query slashing signing-info $(sided tendermint show-validator)
```

List of active validators

```bash
sided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
sided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
sided q staking validator $(sided keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
sided tx distribution withdraw-all-rewards --from <YOUR_WALLET> --fees=5000uside -y
```

Get rewards and commissions from your validator

```bash
sided tx distribution withdraw-rewards $(sided keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --fees=5000uside -y
```

Delegate tokens to your validator

```bash
sided tx staking delegate $(sided keys show <YOUR_WALLET_NAME> --bech val -a) 1000000uside --from <YOUR_WALLET> --fees=5000uside -y
```

Delegate tokens to validator

```bash
sided tx staking delegate <VALOPER_ADDRESS> 1000000uside --from <YOUR_WALLET> --fees=5000uside -y
```

Redelegate tokens to another validator

```bash
sided tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000uside --from <WALLET> --fees=5000uside -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
sided tx staking unbond <VALOPER_ADDRESS> 1000000uside --from <YOUR_WALLET> --fees=5000uside -y
```

Send tokens to another wallet

```bash
sided tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000uside --from <YOUR_WALLET_ADDRESS> --fees=5000uside -y
```

Check info about transaction by hash **TX\_HASH**

```bash
sided query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
sided tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000uside \
--type="Text" \
--from=<WALLET_ADDRESS> \
--fees=5000uside \
-y
```

List of all proposals

```bash
sided query gov proposals
```

Check proposal info by proposal id

```bash
sided query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
sided tx gov deposit 1 1000000uside --from <YOUR_WALLET> --fees=5000uside -y
```

Vote as, **YES**

```bash
sided tx gov vote 1 yes --from <YOUR_WALLET> --fees=5000uside -y
```

Vote as, **NO**

```bash
sided tx gov vote 1 no --from <YOUR_WALLET> --fees=5000uside -y
```

Vote as, **NO\_WITH\_VETO**

```bash
sided tx gov vote 1 no_with_veto --from <YOUR_WALLET> --fees=5000uside -y
```

Vote as, **ABSTAIN**

```bash
sided tx gov vote 1 abstain --from <YOUR_WALLET> --fees=5000uside -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.side/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.side/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.side/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.side/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.side/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.side/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(sided tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.side/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.side/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
sided status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
sided status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
sided status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
sided tendermint unsafe-reset-all --home $HOME/.side --keep-addr-book
```

Delete node

```bash
sudo systemctl stop sided && \
sudo systemctl disable sided && \
sudo rm /etc/systemd/system/sided.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.side && \
rm -rf $HOME/sidechain
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
sided q staking params
sided q slashing params
```

Check validator private key is correct

```bash
[[ $(sided q staking validator $(sided keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(sided status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
sided q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
sided q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable sided
```

Disable service

```bash
sudo systemctl disable sided
```

Start service

```bash
sudo systemctl start sided
```

Stop service

```bash
sudo systemctl stop sided
```

Restart service

```bash
sudo systemctl restart sided
```

Check service status

```bash
sudo systemctl status sided
```

Check service logs

```bash
sudo journalctl -u sided -f -o cat
```
