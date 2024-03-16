---
cover: ../../.gitbook/assets/juno-banner.jpeg
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
sudo systemctl stop junod
cp $HOME/.juno/data/priv_validator_state.json $HOME/.juno/priv_validator_state.json.backup
rm -rf $HOME/.juno/data

URL=https://snapshots-testnet.stake-town.com/juno/uni-6_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.juno

mv $HOME/.juno/priv_validator_state.json.backup $HOME/.juno/data/priv_validator_state.json

sudo systemctl restart junod && sudo journalctl -u junod -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop junod

cp $HOME/.juno/data/priv_validator_state.json $HOME/.juno/priv_validator_state.json.backup
junod tendermint unsafe-reset-all --home $HOME/.juno --keep-addr-book

SNAP_RPC="https://juno-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="5e7b8dda11127e5a08d3480cf763849ef206de1a@65.108.124.43:33656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.juno/config/config.toml

CONFIG_TOML=$HOME/.juno/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.juno/priv_validator_state.json.backup $HOME/.juno/data/priv_validator_state.json

sudo systemctl restart junod && sudo journalctl -u junod -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/juno/addrbook.json > $HOME/.juno/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/juno/genesis.json > $HOME/.juno/config/genesis.json
```
