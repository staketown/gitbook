---
cover: ../../.gitbook/assets/mantra-banner.jpeg
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
sudo systemctl stop mantrachaind
cp $HOME/.mantrachain/data/priv_validator_state.json $HOME/.mantrachain/priv_validator_state.json.backup
rm -rf $HOME/.mantrachain/data

URL=https://snapshots-testnet.stake-town.com/mantra/mantrachain-testnet-1_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.mantrachain

mv $HOME/.mantrachain/priv_validator_state.json.backup $HOME/.mantrachain/data/priv_validator_state.json 

sudo systemctl restart mantrachaind && sudo journalctl -u mantrachaind -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop mantrachaind

cp $HOME/.mantrachain/data/priv_validator_state.json $HOME/.mantrachain/priv_validator_state.json.backup
mantrachaind tendermint unsafe-reset-all --home $HOME/.mantrachain --keep-addr-book

SNAP_RPC="https://mantra-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="eaeb4872c88fa5d3a3fea9deb93cecb552dbe7a3@65.109.65.248:47656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.mantrachain/config/config.toml

CONFIG_TOML=$HOME/.mantrachain/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.mantrachain/priv_validator_state.json.backup $HOME/.mantrachain/data/priv_validator_state.json

sudo systemctl restart mantrachaind && sudo journalctl -u mantrachaind -f -o cat
```

## **Wasm**

As far state-sync doesn't support wasm folder we should download it manually

```bash
URL=https://snapshots-testnet.stake-town.com/mantra/wasm_latest.tar.lz4
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.mantrachain
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/mantra/addrbook.json > $HOME/.mantrachain/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/mantra/genesis.json > $HOME/.mantrachain/config/genesis.json
```
