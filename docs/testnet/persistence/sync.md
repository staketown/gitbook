---
cover: ../../.gitbook/assets/persistence-banner.png
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
sudo systemctl stop persistenceCore
cp $HOME/.persistenceCore/data/priv_validator_state.json $HOME/.persistenceCore/priv_validator_state.json.backup
rm -rf $HOME/.persistenceCore/data

URL=https://snapshots-testnet.stake-town.com/persistence/test-core-2_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.persistenceCore

mv $HOME/.persistenceCore/priv_validator_state.json.backup $HOME/.persistenceCore/data/priv_validator_state.json 

sudo systemctl restart persistenceCore && sudo journalctl -u persistenceCore -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop persistenceCore

cp $HOME/.persistenceCore/data/priv_validator_state.json $HOME/.persistenceCore/priv_validator_state.json.backup
persistenceCore tendermint unsafe-reset-all --home $HOME/.persistenceCore --keep-addr-book

SNAP_RPC="https://persistence-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="d565e0d3af46f5bf2a98411fab5e48498bbad3da@65.108.203.61:53656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.persistenceCore/config/config.toml

CONFIG_TOML=$HOME/.persistenceCore/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.persistenceCore/priv_validator_state.json.backup $HOME/.persistenceCore/data/priv_validator_state.json

sudo systemctl restart persistenceCore && sudo journalctl -u persistenceCore -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots-testnet.stake-town.com/persistence/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.persistenceCore
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/persistence/addrbook.json > $HOME/.persistenceCore/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/persistence/genesis.json > $HOME/.persistenceCore/config/genesis.json
```
