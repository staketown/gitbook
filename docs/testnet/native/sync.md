---
cover: ../../.gitbook/assets/native-banner.jpeg
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
sudo systemctl stop gonatived
cp $HOME/.gonative/data/priv_validator_state.json $HOME/.gonative/priv_validator_state.json.backup
rm -rf $HOME/.gonative/data

URL=https://snapshots-testnet.stake-town.com/native/native-t1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.gonative

mv $HOME/.gonative/priv_validator_state.json.backup $HOME/.gonative/data/priv_validator_state.json

sudo systemctl restart gonatived && sudo journalctl -u gonatived -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop gonatived

cp $HOME/.gonative/data/priv_validator_state.json $HOME/.gonative/priv_validator_state.json.backup
gonative comet unsafe-reset-all --home $HOME/.gonative --keep-addr-book

SNAP_RPC="https://native-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="236946946eacbf6ab8a6f15c99dac1c80db6f8a5@65.108.203.61:52656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.gonative/config/config.toml

CONFIG_TOML=$HOME/.gonative/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.gonative/priv_validator_state.json.backup $HOME/.gonative/data/priv_validator_state.json

sudo systemctl restart gonatived && sudo journalctl -u gonatived -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/native/addrbook.json > $HOME/.gonative/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/native/genesis.json > $HOME/.gonative/config/genesis.json
```
