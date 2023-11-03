---
cover: ../../.gitbook/assets/evmos-banner.png
coverY: 0
---

# Sync

## **Snapshot**

```bash
We don't support snapshots
```

## **State Sync**

```bash
sudo systemctl stop evmosd

cp $HOME/.evmosd/data/priv_validator_state.json $HOME/.evmosd/priv_validator_state.json.backup
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book

SNAP_RPC="https://evmos-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="c8e2800e5743a1575fd8a0fcbb7a74c6f67a23a9@88.99.208.54:40656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.evmosd/config/config.toml

CONFIG_TOML=$HOME/.evmosd/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.evmosd/priv_validator_state.json.backup $HOME/.evmosd/data/priv_validator_state.json

sudo systemctl evmosd archwayd && sudo journalctl -u evmosd -f -o cat
```

## **Address Book**

```bash
we don't support
```

## Genesis

```bash
we don't support
```
