---
cover: ../../.gitbook/assets/elys-banner.jpeg
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
sudo systemctl stop elysd
cp $HOME/.elys/data/priv_validator_state.json $HOME/.elys/priv_validator_state.json.backup
rm -rf $HOME/.elys/data

URL=https://snapshots-testnet.stake-town.com/elys/elysicstestnet-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.elys

mv $HOME/.elys/priv_validator_state.json.backup $HOME/.elys/data/priv_validator_state.json

sudo systemctl restart elysd && sudo journalctl -u elysd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop elysd

cp $HOME/.elys/data/priv_validator_state.json $HOME/.elys/priv_validator_state.json.backup
elysd tendermint unsafe-reset-all --home $HOME/.elys --keep-addr-book

SNAP_RPC="https://elys-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="b64643dc38d426362a4f7c98b6acabe37ffb5654@65.108.203.61:38656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.elys/config/config.toml

CONFIG_TOML=$HOME/.elys/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.elys/priv_validator_state.json.backup $HOME/.elys/data/priv_validator_state.json

sudo systemctl restart elysd && sudo journalctl -u elysd -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/elys/addrbook.json > $HOME/.elys/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/elys/genesis.json > $HOME/.elys/config/genesis.json
```
