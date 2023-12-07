---
cover: ../../.gitbook/assets/umee-banner.png
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
sudo systemctl stop umeed
cp $HOME/.umee/data/priv_validator_state.json $HOME/.umee/priv_validator_state.json.backup
rm -rf $HOME/.umee/data

URL=https://snapshots-testnet.stake-town.com/umee/canon-4_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.umee

mv $HOME/.umee/priv_validator_state.json.backup $HOME/.umee/data/priv_validator_state.json

sudo systemctl restart umeed && sudo journalctl -u umeed -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop umeed

cp $HOME/.umee/data/priv_validator_state.json $HOME/.umee/priv_validator_state.json.backup
umeed tendermint unsafe-reset-all --home $HOME/.umee --keep-addr-book

SNAP_RPC="https://umee-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="56d3b17ccbd6c095a2eb39462101545ca687901d@65.109.65.248:29656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.umee/config/config.toml

CONFIG_TOML=$HOME/.umee/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.umee/priv_validator_state.json.backup $HOME/.umee/data/priv_validator_state.json

sudo systemctl restart umeed && sudo journalctl -u umeed -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/umee/addrbook.json > $HOME/.umee/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/umee/genesis.json > $HOME/.umee/config/genesis.json
```
