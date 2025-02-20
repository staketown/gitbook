---
cover: ../../.gitbook/assets/kyve-banner.jpeg
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
sudo systemctl stop kyved
cp $HOME/.kyve/data/priv_validator_state.json $HOME/.kyve/priv_validator_state.json.backup
rm -rf $HOME/.kyve/data

URL=https://snapshots-1.stake-town.com/kyve/kyve-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.kyve

mv $HOME/.kyve/priv_validator_state.json.backup $HOME/.kyve/data/priv_validator_state.json

sudo systemctl restart kyved && sudo journalctl -u kyved -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop kyved

cp $HOME/.kyve/data/priv_validator_state.json $HOME/.kyve/priv_validator_state.json.backup
kyved tendermint unsafe-reset-all --home $HOME/.kyve --keep-addr-book

SNAP_RPC="https://kyve-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="24feaba7bf73a2e80d4bec88c1004b9377e28495@65.108.195.213:57656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.kyve/config/config.toml

CONFIG_TOML=$HOME/.kyve/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.kyve/priv_validator_state.json.backup $HOME/.kyve/data/priv_validator_state.json

sudo systemctl restart kyved && sudo journalctl -u kyved -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-1.stake-town.com/kyve/addrbook.json > $HOME/.kyve/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-1.stake-town.com/kyve/genesis.json > $HOME/.kyve/config/genesis.json
```
