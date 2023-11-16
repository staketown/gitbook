---
cover: ../../.gitbook/assets/aura-banner.jpeg
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
sudo systemctl stop aurad
cp $HOME/.aura/data/priv_validator_state.json $HOME/.aura/priv_validator_state.json.backup
rm -rf $HOME/.aura/data

URL=https://snapshots.stake-town.com/aura/xstaxy-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.aura

mv $HOME/.aura/priv_validator_state.json.backup $HOME/.aura/data/priv_validator_state.json 

sudo systemctl restart aurad && sudo journalctl -u aurad -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop aurad

cp $HOME/.aura/data/priv_validator_state.json $HOME/.aura/priv_validator_state.json.backup
aurad tendermint unsafe-reset-all --home $HOME/.aura --keep-addr-book

SNAP_RPC="https://aura-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="2bd24dbac94c3b95707c8f06a57d61a31ae666fa@88.99.208.54:46656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.aura/config/config.toml

CONFIG_TOML=$HOME/.aura/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.aura/priv_validator_state.json.backup $HOME/.aura/data/priv_validator_state.json

sudo systemctl restart aurad && sudo journalctl -u aurad -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots.stake-town.com/aura/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.aura
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/aura/addrbook.json > $HOME/.aura/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/aura/genesis.json > $HOME/.aura/config/genesis.json
```
