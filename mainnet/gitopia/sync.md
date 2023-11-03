---
cover: ../../.gitbook/assets/gitopia-banner.png
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
sudo systemctl stop gitopiad
cp $HOME/.gitopia/data/priv_validator_state.json $HOME/.gitopia/priv_validator_state.json.backup
rm -rf $HOME/.gitopia/data

URL="https://snapshots.stake-town.com/gitopia/gitopia_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.gitopia

mv $HOME/.gitopia/priv_validator_state.json.backup $HOME/.gitopia/data/priv_validator_state.json

sudo systemctl restart gitopiad && sudo journalctl -u gitopiad -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop gitopiad

cp $HOME/.gitopia/data/priv_validator_state.json $HOME/.gitopia/priv_validator_state.json.backup
gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia --keep-addr-book

SNAP_RPC="https://gitopia-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="d5525675ceb88d2c4f4df828ec01d237bcc11950@88.99.208.54:26656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.gitopia/config/config.toml

CONFIG_TOML=$HOME/.gitopia/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.gitopia/priv_validator_state.json.backup $HOME/.gitopia/data/priv_validator_state.json

sudo systemctl restart gitopiad && sudo journalctl -u gitopiad -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/gitopia/addrbook.json > $HOME/.gitopia/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/gitopia/genesis.json > $HOME/.gitopia/config/genesis.json
```
