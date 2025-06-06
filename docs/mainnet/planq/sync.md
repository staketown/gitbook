---
cover: ../../.gitbook/assets/planq-banner.jpeg
coverY: 0
---

# Sync

## **Snapshot**

> updated every 6 hours, starting from 00:00 UTC

```bash
# install dependencies, if needed
sudo apt update && sudo apt install lz4 -y
```

```bash
sudo systemctl stop planqd
cp $HOME/.planqd/data/priv_validator_state.json $HOME/.planqd/priv_validator_state.json.backup
rm -rf $HOME/.planqd/data

URL=https://snapshots.stake-town.com/planq/planq_7070-2_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.planqd

mv $HOME/.planqd/priv_validator_state.json.backup $HOME/.planqd/data/priv_validator_state.json

sudo systemctl restart planqd && sudo journalctl -u planqd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop planqd

cp $HOME/.planqd/data/priv_validator_state.json $HOME/.planqd/priv_validator_state.json.backup
planqd tendermint unsafe-reset-all --home $HOME/.planqd --keep-addr-book

SNAP_RPC="https://planq-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="bf85bba541296abdee74aed59c3d660650699403@65.108.129.253:51656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.planqd/config/config.toml

CONFIG_TOML=$HOME/.planqd/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.planqd/priv_validator_state.json.backup $HOME/.planqd/data/priv_validator_state.json

sudo systemctl restart planqd && sudo journalctl -u planqd -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/planq/addrbook.json > $HOME/.planqd/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/planq/genesis.json > $HOME/.planqd/config/genesis.json
```
