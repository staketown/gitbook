---
cover: ../../.gitbook/assets/nolus-defi-banner.png
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
sudo systemctl stop nolusd
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
rm -rf $HOME/.nolus/data

URL="https://snapshots-testnet.stake-town.com/nolus/nolus-rila_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.nolus

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json

sudo systemctl restart nolusd && sudo journalctl -u nolusd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop nolusd

cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book

SNAP_RPC="https://nolus-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="a70d47079283e8bddc0d2c63256b34302f9a0a2b@65.109.65.248:31656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nolus/config/config.toml

CONFIG_TOML=$HOME/.nolus/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json

sudo systemctl restart nolusd && sudo journalctl -u nolusd -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/nolus/addrbook.json > $HOME/.nolus/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/nolus/genesis.json > $HOME/.nolus/config/genesis.json
```
