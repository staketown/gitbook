---
cover: ../../.gitbook/assets/{{BANNER_NAME}}
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
{{BINARY}} keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
{{BINARY}} keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
{{BINARY}} keys list
```

Delete wallet

```bash
{{BINARY}} keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
{{BINARY}} keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
{{BINARY}} keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
{{BINARY}} q bank balances $({{BINARY}} keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
{{BINARY}} tx staking create-validator \
--amount={{AMOUNT}} \
--pubkey=$({{BINARY}} comet show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id={{CHAIN_ID}} \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees={{FEES}} \
-y
```

Edit validator

```bash
{{BINARY}} tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--fees={{FEES}} \
-y
```

Unjail your validator

```bash
{{BINARY}} tx slashing unjail --from <YOUR_WALLET> --fees={{FEES}} -y
```

Check blocks info processed by your validator

```bash
{{BINARY}} query slashing signing-info $({{BINARY}} tendermint show-validator)
```

List of active validators

```bash
{{BINARY}} q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
{{BINARY}} q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
{{BINARY}} q staking validator $({{BINARY}} keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
{{BINARY}} tx distribution withdraw-all-rewards --from <YOUR_WALLET> --fees={{FEES}} -y
```

Get rewards and commissions from your validator

```bash
{{BINARY}} tx distribution withdraw-rewards $({{BINARY}} keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --fees={{FEES}} -y
```

Delegate tokens to your validator

```bash
{{BINARY}} tx staking delegate $({{BINARY}} keys show <YOUR_WALLET_NAME> --bech val -a) {{AMOUNT}} --from <YOUR_WALLET> --fees={{FEES}} -y
```

Delegate tokens to validator

```bash
{{BINARY}} tx staking delegate <VALOPER_ADDRESS> {{AMOUNT}} --from <YOUR_WALLET> --fees={{FEES}} -y
```

Redelegate tokens to another validator

```bash
{{BINARY}} tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> {{AMOUNT}} --from <WALLET> --fees={{FEES}} -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
{{BINARY}} tx staking unbond <VALOPER_ADDRESS> {{AMOUNT}} --from <YOUR_WALLET> --fees={{FEES}} -y
```

Send tokens to another wallet

```bash
{{BINARY}} tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> {{AMOUNT}} --from <YOUR_WALLET_ADDRESS> --fees={{FEES}} -y
```

Check info about transaction by hash **TX\_HASH**

```bash
{{BINARY}} query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
{{BINARY}} tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit={{AMOUNT}} \
--type="Text" \
--from=<WALLET_ADDRESS> \
--fees={{FEES}} \
-y
```

List of all proposals

```bash
{{BINARY}} query gov proposals
```

Check proposal info by proposal id

```bash
{{BINARY}} query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
{{BINARY}} tx gov deposit 1 {{AMOUNT}} --from <YOUR_WALLET> --fees={{FEES}} -y
```

Vote as, **YES**

```bash
{{BINARY}} tx gov vote 1 yes --from <YOUR_WALLET> --fees={{FEES}} -y
```

Vote as, **NO**

```bash
{{BINARY}} tx gov vote 1 no --from <YOUR_WALLET> --fees={{FEES}} -y
```

Vote as, **NO\_WITH\_VETO**

```bash
{{BINARY}} tx gov vote 1 no_with_veto --from <YOUR_WALLET> --fees={{FEES}} -y
```

Vote as, **ABSTAIN**

```bash
{{BINARY}} tx gov vote 1 abstain --from <YOUR_WALLET> --fees={{FEES}} -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/{{WORKING_DIR}}/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/{{WORKING_DIR}}/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/{{WORKING_DIR}}/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/{{WORKING_DIR}}/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/{{WORKING_DIR}}/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/{{WORKING_DIR}}/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $({{BINARY}} comet show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/{{WORKING_DIR}}/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/{{WORKING_DIR}}/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
{{BINARY}} comet status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
{{BINARY}} comet status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
{{BINARY}} comet status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
{{BINARY}} comet unsafe-reset-all --home $HOME/{{WORKING_DIR}} --keep-addr-book
```

Delete node

```bash
sudo systemctl stop {{SERVICE}} && \
sudo systemctl disable {{SERVICE}} && \
sudo rm /etc/systemd/system/{{SERVICE}}.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/{{WORKING_DIR}} && \
rm -rf $HOME/{{PROJECT_DIR}}
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
{{BINARY}} q staking params
{{BINARY}} q slashing params
```

Check validator private key is correct

```bash
[[ $({{BINARY}} q staking validator $({{BINARY}} keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $({{BINARY}} status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
{{BINARY}} q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
{{BINARY}} q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable {{SERVICE}}
```

Disable service

```bash
sudo systemctl disable {{SERVICE}}
```

Start service

```bash
sudo systemctl start {{SERVICE}}
```

Stop service

```bash
sudo systemctl stop {{SERVICE}}
```

Restart service

```bash
sudo systemctl restart {{SERVICE}}
```

Check service status

```bash
sudo systemctl status {{SERVICE}}
```

Check service logs

```bash
sudo journalctl -u {{SERVICE}} -f -o cat
```
