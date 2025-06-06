---
cover: ../../.gitbook/assets/picasso-banner.jpeg
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
sudo systemctl stop picad
cp $HOME/.banksy/data/priv_validator_state.json $HOME/.banksy/priv_validator_state.json.backup
rm -rf $HOME/.banksy/data

URL=https://snapshots.stake-town.com/picasso/centauri-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.banksy

mv $HOME/.banksy/priv_validator_state.json.backup $HOME/.banksy/data/priv_validator_state.json 

sudo systemctl restart picad && sudo journalctl -u picad -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop picad

cp $HOME/.banksy/data/priv_validator_state.json $HOME/.banksy/priv_validator_state.json.backup
picad tendermint unsafe-reset-all --home $HOME/.banksy --keep-addr-book

SNAP_RPC="https://picasso-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="f36ea8178f94baf30268cf669caab0d8b314c08b@65.108.129.253:37656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.banksy/config/config.toml

CONFIG_TOML=$HOME/.banksy/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.banksy/priv_validator_state.json.backup $HOME/.banksy/data/priv_validator_state.json

sudo systemctl restart picad && sudo journalctl -u picad -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots.stake-town.com/picasso/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.banksy
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/picasso/addrbook.json > $HOME/.banksy/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/picasso/genesis.json > $HOME/.banksy/config/genesis.json
```
