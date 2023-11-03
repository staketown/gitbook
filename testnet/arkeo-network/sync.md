---
cover: ../../.gitbook/assets/arkeo-banner.jpeg
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
sudo systemctl stop arkeod
cp $HOME/.arkeo/data/priv_validator_state.json $HOME/.arkeo/priv_validator_state.json.backup
rm -rf $HOME/.arkeo/data

URL="https://snapshots-testnet.stake-town.com/arkeo/arkeo_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.arkeo

mv $HOME/.arkeo/priv_validator_state.json.backup $HOME/.arkeo/data/priv_validator_state.json

sudo systemctl restart arkeod && sudo journalctl -u arkeod -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop arkeod

cp $HOME/.arkeo/data/priv_validator_state.json $HOME/.arkeo/priv_validator_state.json.backup
arkeod tendermint unsafe-reset-all --home $HOME/.arkeo --keep-addr-book

SNAP_RPC="https://arkeo-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="e6b058d1d6be000d67b87e9d11cb0de1bba1e477@65.109.65.248:42656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.archway/config/config.toml

CONFIG_TOML=$HOME/.arkeo/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.arkeo/priv_validator_state.json.backup $HOME/.arkeo/data/priv_validator_state.json

sudo systemctl restart arkeod && sudo journalctl -u arkeod -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/arkeo/addrbook.json > $HOME/.archway/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/arkeo/genesis.json > $HOME/.archway/config/genesis.json
```
