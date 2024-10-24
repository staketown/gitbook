---
cover: ../../.gitbook/assets/prysm-banner.png
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
sudo systemctl stop prysmd
cp $HOME/.prysm/data/priv_validator_state.json $HOME/.prysm/priv_validator_state.json.backup
rm -rf $HOME/.prysm/data

URL=https://snapshots-testnet.stake-town.com/prysm/prysm-devnet-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.prysm

mv $HOME/.prysm/priv_validator_state.json.backup $HOME/.prysm/data/priv_validator_state.json 

sudo systemctl restart prysmd && sudo journalctl -u prysmd -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop prysmd

cp $HOME/.prysm/data/priv_validator_state.json $HOME/.prysm/priv_validator_state.json.backup
prysmd tendermint unsafe-reset-all --home $HOME/.prysm --keep-addr-book

SNAP_RPC="https://prysm-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="69509925a520c5c7c5f505ec4cedab95073388e5@136.243.13.36:29856,bc1a37c7656e6f869a01bb8dabaf9ca58fe61b0c@5.9.73.170:29856,b377fd0b14816eef8e12644340845c127d1e7d93@79.13.87.34:26656,c80143f844fd8da4f76a0a43de86936f72372168@184.107.57.137:18656,afc7a20c15bde738e68781238307f4481938109d@94.130.35.120:18656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.prysm/config/config.toml

CONFIG_TOML=$HOME/.prysm/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.prysm/priv_validator_state.json.backup $HOME/.prysm/data/priv_validator_state.json

sudo systemctl restart prysmd && sudo journalctl -u prysmd -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots-testnet.stake-town.com/prysm/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.prysm
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/prysm/addrbook.json > $HOME/.prysm/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/prysm/genesis.json > $HOME/.prysm/config/genesis.json
```
