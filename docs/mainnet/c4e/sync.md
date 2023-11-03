---
cover: ../../.gitbook/assets/c4e-banner.jpeg
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
sudo systemctl stop c4ed
cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
rm -rf $HOME/.c4e-chain/data

URL="https://snapshots.stake-town.com/c4e/perun-1_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.c4e-chain

mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json

sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop c4ed

cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book

SNAP_RPC="https://c4e-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="dd3e39a35cc96bae4b51c68605237dafb6de284c@88.99.208.54:35656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.c4e-chain/config/config.toml

CONFIG_TOML=$HOME/.c4e-chain/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json

sudo systemctl restart c4ed
sudo journalctl -u c4ed -f -o cat
```

## Genesis

```bash
curl -s https://snapshots.stake-town.com/c4e/genesis.json > $HOME/.c4e-chain/config/genesis.json
```
