---
cover: ../../.gitbook/assets/celestia-banner.jpeg
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
sudo systemctl stop celestia-appd
cp $HOME/.celestia-app/data/priv_validator_state.json $HOME/.celestia-app/priv_validator_state.json.backup
rm -rf $HOME/.celestia-app/data

URL=https://snapshots-testnet.stake-town.com/celestia/mocha-4_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.celestia-app

mv $HOME/.celestia-app/priv_validator_state.json.backup $HOME/.celestia-app/data/priv_validator_state.json

sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop celestia-appd

cp $HOME/.celestia-app/data/priv_validator_state.json $HOME/.celestia-app/priv_validator_state.json.backup
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app --keep-addr-book

SNAP_RPC="https://celestia-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="94035b3187ebfeeadffdae4dc69292ad06b581a2@65.109.92.217:34656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.celestia-app/config/config.toml

CONFIG_TOML=$HOME/.celestia-app/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.celestia-app/priv_validator_state.json.backup $HOME/.celestia-app/data/priv_validator_state.json

sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/celestia/addrbook.json > $HOME/.celestia-app/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/celestia/genesis.json > $HOME/.celestia-app/config/genesis.json
```
