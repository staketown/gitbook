---
cover: ../../.gitbook/assets/0g-banner.jpg
coverY: 0
---

# Cheat Sheet

### Wallet operations

Create wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
0gchaind keys add <YOUR_WALLET_NAME> --eth
```

Recover wallet

> ⚠️ store **seed** phrase, important during recovering

```bash
0gchaind keys add <YOUR_WALLET_NAME> --eth --recover
```

List of all wallets

```bash
0gchaind keys list
```

Delete wallet

```bash
0gchaind keys delete <YOUR_WALLET_NAME>
```

Export wallet

> ⚠️ save to wallet.backup

```bash
0gchaind keys export <YOUR_WALLET_NAME>
```

Import wallet

```bash
0gchaind keys import <WALLET_NAME> wallet.backup
```

Check wallet balance

```bash
0gchaind q bank balances $(0gchaind keys show <YOUR_WALLET_NAME> -a)
```

### Validator operations

Create validator

```bash
0gchaind tx staking create-validator \
--amount=1000000ua0gi \
--pubkey=$(0gchaind tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<Your identity> \
--details="<Your details>" \
--chain-id=zgtendermint_16600-2 \
--commission-rate=0.05 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.1 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=1000ua0gi \
-y
```

Edit validator

```bash
0gchaind tx staking edit-validator \
--new-moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--commission-rate=0.1 \
--from=<YOUR_WALLET> \
--fees=1000ua0gi \
-y
```

Unjail your validator

```bash
0gchaind tx slashing unjail --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Check blocks info processed by your validator

```bash
0gchaind query slashing signing-info $(0gchaind tendermint show-validator)
```

List of active validators

```bash
0gchaind q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List of inactive validators

```bash
0gchaind q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED") or .status=="BOND_STATUS_UNBONDING")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Info about your validator

```bash
0gchaind q staking validator $(0gchaind keys show <YOUR_WALLET_NAME> --bech val -a)
```

### Transactions

Get your rewards from all validators

```bash
0gchaind tx distribution withdraw-all-rewards --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Get rewards and commissions from your validator

```bash
0gchaind tx distribution withdraw-rewards $(0gchaind keys show <YOUR_WALLET_NAME> --bech val -a) --commission --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Delegate tokens to your validator

```bash
0gchaind tx staking delegate $(0gchaind keys show <YOUR_WALLET_NAME> --bech val -a) 1000000ua0gi --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Delegate tokens to validator

```bash
0gchaind tx staking delegate <VALOPER_ADDRESS> 1000000ua0gi --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Redelegate tokens to another validator

```bash
0gchaind tx staking redelegate <SRC_VALOPER_ADDRESS> <TARGET_VALOPER_ADDRESS> 1000000ua0gi --from <WALLET> --fees=1000ua0gi -y
```

Unbound tokens from validator

> ⚠️ it’s can take a while, \~21 days, depends on network’s parameters

```bash
0gchaind tx staking unbond <VALOPER_ADDRESS> 1000000ua0gi --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Send tokens to another wallet

```bash
0gchaind tx bank send <YOUR_WALLET_ADDRESS> <TARGET_WALLET_ADDRESS> 1000000ua0gi --from <YOUR_WALLET_ADDRESS> --fees=1000ua0gi -y
```

Check info about transaction by hash **TX\_HASH**

```bash
0gchaind query tx <TX_HASH>
```

### Governance

Submit text proposal

```bash
0gchaind tx gov submit-proposal \
--title="<Your Title>" \
--description="<Your Description>" \
--deposit=1000000ua0gi \
--type="Text" \
--from=<WALLET_ADDRESS> \
--fees=1000ua0gi \
-y
```

List of all proposals

```bash
0gchaind query gov proposals
```

Check proposal info by proposal id

```bash
0gchaind query gov proposal <proposal_id>
```

Deposit proposal by proposal id

```bash
0gchaind tx gov deposit 1 1000000ua0gi --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Vote as, **YES**

```bash
0gchaind tx gov vote 1 yes --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Vote as, **NO**

```bash
0gchaind tx gov vote 1 no --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Vote as, **NO\_WITH\_VETO**

```bash
0gchaind tx gov vote 1 no_with_veto --from <YOUR_WALLET> --fees=1000ua0gi -y
```

Vote as, **ABSTAIN**

```bash
0gchaind tx gov vote 1 abstain --from <YOUR_WALLET> --fees=1000ua0gi -y
```

### Utils

Change ports to custom

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:7060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.0gchain/config/config.toml && \
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:10090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:10091\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:2317\"%" $HOME/.0gchain/config/app.toml && \
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.0gchain/config/client.toml
```

Turn on indexing

```bash
sed -i 's|^indexer *=.*|indexer = "kv"|' $HOME/.0gchain/config/config.toml
```

Turn off indexing

```bash
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.0gchain/config/config.toml
```

Setup custom prunning

```bash
APP_TOML="~/.0gchain/config/app.toml"
sed -i 's|^pruning *=.*|pruning = "custom"|' $APP_TOML
sed -i 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' $APP_TOML
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $APP_TOML
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $APP_TOML
```

Check your peer

```bash
echo $(0gchaind tendermint show-node-id)@$(curl http://ifconfig.me/)$(grep -A 3 "\[p2p\]" ~/.0gchain/config/config.toml | egrep -o ":[0-9]+")
```

Check your RPC

```bash
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.0gchain/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

Check information about validator

```bash
0gchaind status 2>&1 | jq .ValidatorInfo
```

Check synchronisation status (**false - synced, true - not synced**)

```bash
0gchaind status 2>&1 | jq .SyncInfo.catching_up
```

Check the latest block

```bash
0gchaind status 2>&1 | jq .SyncInfo.latest_block_height
```

Reset network

```bash
0gchaind tendermint unsafe-reset-all --home $HOME/.0gchain --keep-addr-book
```

Delete node

```bash
sudo systemctl stop 0gchaind && \
sudo systemctl disable 0gchaind && \
sudo rm /etc/systemd/system/0gchaind.service && \
sudo systemctl daemon-reload && \
rm -rf $HOME/.0gchain && \
rm -rf $HOME/0g-chain
```

Check IP address of the server

```bash
wget -qO- eth0.me
```

Check network parameters

```bash
0gchaind q staking params
0gchaind q slashing params
```

Check validator private key is correct

```bash
[[ $(0gchaind q staking validator $(0gchaind keys show <YOUR_WALLET> --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(0gchaind status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

Search all output transactions by address

```bash
0gchaind q txs --events transfer.sender=<ADDRESS> 2>&1 | jq | grep txhash
```

Search all input transactions by address

```bash
0gchaind q txs --events transfer.recipient=<ADDRESS> 2>&1 | jq | grep txhash
```

### Service management

Reload services

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable 0gchaind
```

Disable service

```bash
sudo systemctl disable 0gchaind
```

Start service

```bash
sudo systemctl start 0gchaind
```

Stop service

```bash
sudo systemctl stop 0gchaind
```

Restart service

```bash
sudo systemctl restart 0gchaind
```

Check service status

```bash
sudo systemctl status 0gchaind
```

Check service logs

```bash
sudo journalctl -u 0gchaind -f -o cat
```
