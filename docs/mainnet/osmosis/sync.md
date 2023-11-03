---
cover: ../../.gitbook/assets/osmosis-banner.png
coverY: 0
---

# Sync

## **State Sync**

```bash
sudo systemctl stop osmosisd

cp $HOME/.osmosisd/data/priv_validator_state.json $HOME/.osmosisd/priv_validator_state.json.backup
osmosisd tendermint unsafe-reset-all --home $HOME/.osmosisd --keep-addr-book

SNAP_RPC="https://osmosis-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="1fd3c5d3bb28bef6615fdd8ab6dc6008df646a87@88.99.208.54:41656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.osmosisd/config/config.toml

CONFIG_TOML=$HOME/.osmosisd/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.osmosisd/priv_validator_state.json.backup $HOME/.osmosisd/data/priv_validator_state.json

sudo systemctl restart osmosisd && sudo journalctl -u osmosisd -f -o cat
```
