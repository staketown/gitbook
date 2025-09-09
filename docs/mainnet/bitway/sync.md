---
cover: ../../.gitbook/assets/bitway-banner.jpeg
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
sudo systemctl stop bitwayd
cp $HOME/.bitway/data/priv_validator_state.json $HOME/.bitway/priv_validator_state.json.backup
rm -rf $HOME/.bitway/data

URL=https://snapshots-1.stake-town.com/bitway/bitway-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.bitway

mv $HOME/.bitway/priv_validator_state.json.backup $HOME/.bitway/data/priv_validator_state.json 

sudo systemctl restart bitwayd && sudo journalctl -u bitwayd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop bitwayd

cp $HOME/.bitway/data/priv_validator_state.json $HOME/.bitway/priv_validator_state.json.backup
bitwayd tendermint unsafe-reset-all --home $HOME/.bitway --keep-addr-book

SNAP_RPC="https://bitway-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="aff78303b6172a4ee0ca87adbae017d5fe27f50a@65.108.195.213:49656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.bitway/config/config.toml

CONFIG_TOML=$HOME/.bitway/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.bitway/priv_validator_state.json.backup $HOME/.bitway/data/priv_validator_state.json

sudo systemctl restart bitwayd && sudo journalctl -u bitwayd -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots-1.stake-town.com/bitway/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.bitway
```

## **Address Book**

```bash
curl -Ls https://snapshots-1.stake-town.com/bitway/addrbook.json > $HOME/.bitway/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-1.stake-town.com/bitway/genesis.json > $HOME/.bitway/config/genesis.json
```
