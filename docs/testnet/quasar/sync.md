---
cover: ../../.gitbook/assets/quasar-banner.jpeg
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
sudo systemctl stop quasard
cp $HOME/.quasarnode/data/priv_validator_state.json $HOME/.quasarnode/priv_validator_state.json.backup
rm -rf $HOME/.quasarnode/data

URL=https://snapshots-testnet.stake-town.com/quasar/quasar-test-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.quasarnode

mv $HOME/.quasarnode/priv_validator_state.json.backup $HOME/.quasarnode/data/priv_validator_state.json 

sudo systemctl restart quasard && sudo journalctl -u quasard -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop quasard

cp $HOME/.quasarnode/data/priv_validator_state.json $HOME/.quasarnode/priv_validator_state.json.backup
quasard tendermint unsafe-reset-all --home $HOME/.quasarnode --keep-addr-book

SNAP_RPC="https://quasar-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="d6ca455a4b55b4409e67ec2e0ba11f09f4afcdaa@65.109.65.248:45656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.quasarnode/config/config.toml

CONFIG_TOML=$HOME/.quasarnode/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.quasarnode/priv_validator_state.json.backup $HOME/.quasarnode/data/priv_validator_state.json

sudo systemctl restart quasard && sudo journalctl -u quasard -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots-testnet.stake-town.com/quasar/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.quasarnode
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/quasar/addrbook.json > $HOME/.quasarnode/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/quasar/genesis.json > $HOME/.quasarnode/config/genesis.json
```
