---
cover: ../../.gitbook/assets/{{BANNER_NAME}}
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
sudo systemctl stop {{BINARY}}
cp $HOME/{{WORKING_DIR}}/data/priv_validator_state.json $HOME/{{WORKING_DIR}}/priv_validator_state.json.backup
rm -rf $HOME/{{WORKING_DIR}}/data

URL=https://snapshots{{SNAP_SUBDIR}}.stake-town.com/{{SCRIPT_DIR}}/{{CHAIN_ID}}_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/{{WORKING_DIR}}

mv $HOME/{{WORKING_DIR}}/priv_validator_state.json.backup $HOME/{{WORKING_DIR}}/data/priv_validator_state.json 

sudo systemctl restart {{BINARY}} && sudo journalctl -u {{BINARY}} -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop {{BINARY}}

cp $HOME/{{WORKING_DIR}}/data/priv_validator_state.json $HOME/{{WORKING_DIR}}/priv_validator_state.json.backup
{{BINARY}} tendermint unsafe-reset-all --home $HOME/{{WORKING_DIR}} --keep-addr-book

SNAP_RPC="https://quasar{{SNAP_SUBDIR}}-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="{{PEERS}}"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/{{WORKING_DIR}}/config/config.toml

CONFIG_TOML=$HOME/{{WORKING_DIR}}/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/{{WORKING_DIR}}/priv_validator_state.json.backup $HOME/{{WORKING_DIR}}/data/priv_validator_state.json

sudo systemctl restart {{BINARY}} && sudo journalctl -u {{BINARY}} -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots{{SNAP_SUBDIR}}.stake-town.com/{{SCRIPT_DIR}}/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/{{WORKING_DIR}}
```

## **Address Book**

```bash
curl -Ls https://snapshots{{SNAP_SUBDIR}}.stake-town.com/{{SCRIPT_DIR}}/addrbook.json > $HOME/{{WORKING_DIR}}/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots{{SNAP_SUBDIR}}.stake-town.com/{{SCRIPT_DIR}}/genesis.json > $HOME/{{WORKING_DIR}}/config/genesis.json
```
