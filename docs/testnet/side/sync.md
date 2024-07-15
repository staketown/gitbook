---
cover: ../../.gitbook/assets/side-banner.png
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
sudo systemctl stop sided
cp $HOME/.side/data/priv_validator_state.json $HOME/.side/priv_validator_state.json.backup
rm -rf $HOME/.side/data

URL=https://snapshots-testnet.stake-town.com/side/grimoria-testnet-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.side

mv $HOME/.side/priv_validator_state.json.backup $HOME/.side/data/priv_validator_state.json 

sudo systemctl restart sided && sudo journalctl -u sided -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop sided

cp $HOME/.side/data/priv_validator_state.json $HOME/.side/priv_validator_state.json.backup
sided tendermint unsafe-reset-all --home $HOME/.side --keep-addr-book

SNAP_RPC="https://side-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="027ef6300590b1ca3a2b92a274247e24537bd9c9@65.109.65.248:49656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.side/config/config.toml

CONFIG_TOML=$HOME/.side/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.side/priv_validator_state.json.backup $HOME/.side/data/priv_validator_state.json

sudo systemctl restart sided && sudo journalctl -u sided -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots-testnet.stake-town.com/side/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.side
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/side/addrbook.json > $HOME/.side/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/side/genesis.json > $HOME/.side/config/genesis.json
```
