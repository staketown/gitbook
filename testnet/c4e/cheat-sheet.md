---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
c4ed keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
c4ed keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
c4ed keys list
```

Delete wallet

```bash
c4ed keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
c4ed keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
c4ed keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
c4ed q bank balances $(c4ed keys show wallet -a)
```

### Validator operations

```bash
c4ed tx staking create-validator \
--amount=1000000uc4e \
--pubkey=$(c4ed tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=babajaga-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.10 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000uc4e
-y
```

Edit validator

```bash
c4ed tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=babajaga-1 \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--fees=5000uc4e
-y
```

Unjail your validator

```bash
c4ed tx slashing unjail --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Check blocks info processed by your validator

```bash
c4ed query slashing signing-info $(c4ed tendermint show-validator)
```

List of active validators

```bash
c4ed q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
c4ed q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
c4ed q staking validator $(c4ed keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
c4ed tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Get rewards and commissions from your validator

```bash
c4ed tx distribution withdraw-rewards $(c4ed keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Delegate tokens to your validator

```bash
c4ed tx staking delegate $(c4ed keys show <YOUR_WALLET_NAME> --bech val -a) 1000000uc4e --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Delegate tokens to validator

```bash
c4ed tx staking delegate <VALOPER_ADDRESS> 1000000uc4e --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Redelegate tokens to another validator

```bash
c4ed tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000uc4e --from <WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Unbound tokens from validator

> ⚠️  it’s can take a while, \~21 days, depends on network’s parameters

```bash
c4ed tx staking unbond <VALOPER_ADDRESS> 1000000uc4e --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Send tokens to another wallet

```bash
c4ed tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000uc4e --from <YOUR_WALLET_ADDRESS> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
c4ed query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
c4ed tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000uc4e \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0.1uc4e \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

List of all proposals

```bash
c4ed query gov proposals
```

Check proposal info by proposal id

```bash
c4ed query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
c4ed tx gov deposit 1 1000000uc4e --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Vote as, **YES**

```bash
c4ed tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Vote as, **NO**

```bash
c4ed tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
c4ed tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
c4ed tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0.1uc4e --gas-adjustment 1.5 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.c4e-chain/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.c4e-chain/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.c4e-chain/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.c4e-chain/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.c4e-chain/config/config.toml
```

Setup custom prunning

```bash
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.c4e-chain/config/app.toml
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $HOME/.c4e-chain/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.c4e-chain/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.c4e-chain/config/app.toml
```

Check your peer

```bash
echo $(c4ed tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.c4e-chain/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.c4e-chain/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
c4ed status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
c4ed status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
c4ed status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book
```

Delete node

```bash
sudo systemctl stop c4ed && \
sudo systemctl disable c4ed && \
sudo rm /etc/systemd/system/c4ed.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.c4e-chain && \
rm -rf $HOME/c4e-chain
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
c4ed q staking params
c4ed q slashing params
```

Check validator private key is correct

```bash
[[ $(c4ed q staking validator $(c4ed keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(c4ed status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
c4ed q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
c4ed q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable c4ed
```

Disable service

```bash
sudo systemctl disable c4ed
```

Start service

```bash
sudo systemctl start c4ed
```

Stop service

```bash
sudo systemctl stop c4ed
```

Restart service

```bash
sudo systemctl restart c4ed
```

Check service status

```bash
sudo systemctl status c4ed
```

Check service logs

```bash
sudo journalctl -u c4ed -f -o cat
```
