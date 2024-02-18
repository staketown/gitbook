---
cover: ../../.gitbook/assets/celestia-banner.jpeg
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
celestia-appd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
celestia-appd keys add <YOUR_WALLET_NAME> --recover
```

List of all wallets

```bash
celestia-appd keys list
```

Delete wallet

```bash
celestia-appd keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
celestia-appd keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
celestia-appd keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
celestia-appd q bank balances $(celestia-appd keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
celestia-appd tx staking create-validator \
--amount=1000000utia \
--pubkey=$(celestia-appd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=mocha-4 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--gas-prices=0.002utia \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Edit validator

```bash
celestia-appd tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--gas-prices=0.002utia \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

Unjail your validator

```bash
celestia-appd tx slashing unjail --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Check blocks info processed by your validator

```bash
celestia-appd query slashing signing-info $(celestia-appd tendermint show-validator)
```

List of active validators

```bash
celestia-appd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
celestia-appd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
celestia-appd q staking validator $(celestia-appd keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
celestia-appd tx distribution withdraw-all-rewards --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Get rewards and commissions from your validator

```bash
celestia-appd tx distribution withdraw-rewards $(celestia-appd keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to your validator

```bash
celestia-appd tx staking delegate $(celestia-appd keys show <YOUR_WALLET_NAME> --bech val -a) 1000000utia --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Delegate tokens to validator

```bash
celestia-appd tx staking delegate <VALOPER_ADDRESS> 1000000utia --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Redelegate tokens to another validator

```bash
celestia-appd tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000utia --from <WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
celestia-appd tx staking unbond <VALOPER_ADDRESS> 1000000utia --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Send tokens to another wallet

```bash
celestia-appd tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000utia --from <YOUR_WALLET_ADDRESS> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Check info about transaction by hash **TX\_HASH**

```bash
celestia-appd query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
celestia-appd tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000utia \
--type="Text" \
--from=<WALLET_ADDRESS> \
--gas-prices=0.002utia \
--gas-adjustment=1.4 \
--gas=auto \
-y
```

List of all proposals

```bash
celestia-appd query gov proposals
```

Check proposal info by proposal id

```bash
celestia-appd query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
celestia-appd tx gov deposit 1 1000000utia --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Vote as, **YES**

```bash
celestia-appd tx gov vote 1 yes --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO**

```bash
celestia-appd tx gov vote 1 no --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Vote as, **NO\_WITH\_VETO**

```bash
celestia-appd tx gov vote 1 no_with_veto --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

Vote as, **ABSTAIN**

```bash
celestia-appd tx gov vote 1 abstain --from <YOUR_WALLET> --gas-prices 0.002utia --gas-adjustment 1.4 --gas auto -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.celestia-app/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.celestia-app/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.celestia-app/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.celestia-app/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.celestia-app/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.celestia-app/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(celestia-appd tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.celestia-app/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.celestia-app/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
celestia-appd status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
celestia-appd status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
celestia-appd status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app --keep-addr-book
```

Delete node

```bash
sudo systemctl stop celestia-appd && \
sudo systemctl disable celestia-appd && \
sudo rm /etc/systemd/system/celestia-appd.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.celestia-app && \
rm -rf $HOME/celestia-app
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
celestia-appd q staking params
celestia-appd q slashing params
```

Check validator private key is correct

```bash
[[ $(celestia-appd q staking validator $(celestia-appd keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(celestia-appd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
celestia-appd q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
celestia-appd q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable celestia-appd
```

Disable service

```bash
sudo systemctl disable celestia-appd
```

Start service

```bash
sudo systemctl start celestia-appd
```

Stop service

```bash
sudo systemctl stop celestia-appd
```

Restart service

```bash
sudo systemctl restart celestia-appd
```

Check service status

```bash
sudo systemctl status celestia-appd
```

Check service logs

```bash
sudo journalctl -u celestia-appd -f -o cat
```

### Bridge node useful commands

Get bridge node ID

```bash
AUTH_TOKEN=$(celestia bridge auth admin --p2p.network mocha)
curl -s -X POST -H "Authorization: Bearer $AUTH_TOKEN" -H 'Content-Type: application/json' -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' http://localhost:12058 | jq -r .result.ID
```

Get bridge node key
```bash
cel-key show bridge-wallet --node.type bridge --p2p.network mocha -a | tail -1
```

Check bridge node wallet balance
```bash
celestia-appd q bank balances $(cel-key show bridge-wallet --node.type bridge --p2p.network mocha -a | tail -1)
```

Check bridge node version
```bash
celestia version
```

Check bridge node logs
```bash
journalctl -u celestia-bridge.service -f -o cat
```

### Orchestrator useful commands

Check the EVM address that is linked to your validator.

```bash
celestia-appd query qgb evm <YOUR_VALIDATOR_ADDRESS>
```
or
```bash
celestia-appd query qgb evm $(celestia-appd keys show wallet --bech val -a)
```

Change validator EVM address
```bash
celestia-appd tx qgb register \
<YOUR_VALIDATOR_ADDRESS> \
<NEW_EVM_ADDRESS> \
--from <YOUR_WALLET_ADDRESS> \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.002utia \
-y
```