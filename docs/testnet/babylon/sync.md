---
cover: ../../.gitbook/assets/babylon-banner.jpeg
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
sudo systemctl stop babylond
cp $HOME/.babylond/data/priv_validator_state.json $HOME/.babylond/priv_validator_state.json.backup
rm -rf $HOME/.babylond/data

URL=https://snapshots-testnet.stake-town.com/babylon/bbn-test-5_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.babylond

mv $HOME/.babylond/priv_validator_state.json.backup $HOME/.babylond/data/priv_validator_state.json 

sudo systemctl restart babylond && sudo journalctl -u babylond -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop babylond

cp $HOME/.babylond/data/priv_validator_state.json $HOME/.babylond/priv_validator_state.json.backup
babylond tendermint unsafe-reset-all --home $HOME/.babylond --keep-addr-book

SNAP_RPC="https://babylon-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="e0a8511a01f33de0dbc02b7b0cc53b3815fb3d7a@65.108.203.61:48656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml

CONFIG_TOML=$HOME/.babylond/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.babylond/priv_validator_state.json.backup $HOME/.babylond/data/priv_validator_state.json

sudo systemctl restart babylond && sudo journalctl -u babylond -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots-testnet.stake-town.com/babylon/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.babylond
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/babylon/addrbook.json > $HOME/.babylond/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/babylon/genesis.json > $HOME/.babylond/config/genesis.json
```
