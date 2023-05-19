---
cover: ../../.gitbook/assets/cascadia-banner.jpeg
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
sudo systemctl stop cascadiad
cp $HOME/.cascadiad/data/priv_validator_state.json $HOME/.cascadiad/priv_validator_state.json.backup
rm -rf $HOME/.cascadiad/data

URL="https://snapshots-testnet.stake-town.com/cascadia/cascadia_6102-1_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.cascadiad

mv $HOME/.cascadiad/priv_validator_state.json.backup $HOME/.cascadiad/data/priv_validator_state.json

sudo systemctl restart cascadiad && sudo journalctl -u cascadiad -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop cascadiad

cp $HOME/.cascadiad/data/priv_validator_state.json $HOME/.cascadiad/priv_validator_state.json.backup
cascadiad tendermint unsafe-reset-all --home $HOME/.cascadiad --keep-addr-book

SNAP_RPC="https://cascadia-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="21ca2712116138429aed3d72422379397c53fa86@65.109.65.248:34656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.cascadiad/config/config.toml

CONFIG_TOML=$HOME/.cascadiad/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.cascadiad/priv_validator_state.json.backup $HOME/.cascadiad/data/priv_validator_state.json

sudo systemctl restart cascadiad
sudo journalctl -u cascadiad -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/cascadia/addrbook.json > $HOME/.cascadiad/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/cascadia/genesis.json > $HOME/.cascadiad/config/genesis.json
```
