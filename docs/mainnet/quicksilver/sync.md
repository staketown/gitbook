---
cover: ../../.gitbook/assets/quicksilver-banner.png
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
sudo systemctl stop quicksilverd
cp $HOME/.quicksilverd/data/priv_validator_state.json $HOME/.quicksilverd/priv_validator_state.json.backup
rm -rf $HOME/.quicksilverd/data

URL=https://snapshots.stake-town.com/quicksilver/quicksilver-2_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.quicksilverd

mv $HOME/.quicksilverd/priv_validator_state.json.backup $HOME/.quicksilverd/data/priv_validator_state.json

sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop quicksilverd

cp $HOME/.quicksilverd/data/priv_validator_state.json $HOME/.quicksilverd/priv_validator_state.json.backup
quicksilverd tendermint unsafe-reset-all --home $HOME/.quicksilverd --keep-addr-book

SNAP_RPC="https://quicksilver-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="2da913aa78101b31fc6aa24bca0ef26c1f9d986c@65.108.129.253:50656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.quicksilverd/config/config.toml

CONFIG_TOML=$HOME/.quicksilverd/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.quicksilverd/priv_validator_state.json.backup $HOME/.quicksilverd/data/priv_validator_state.json

sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots.stake-town.com/quicksilver/addrbook.json > $HOME/.quicksilverd/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots.stake-town.com/quicksilver/genesis.json > $HOME/.quicksilverd/config/genesis.json
```
