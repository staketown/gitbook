---
cover: ../../.gitbook/assets/c4e-banner.jpeg
coverY: 0
---

# Sync

## **State Sync**

```bash
sudo systemctl stop c4ed

cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book

SNAP_RPC="https://c4e-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="acbe09ef95434a0cd5547dcbc7da3bb1520d16ee@65.109.65.248:35656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.cascadiad/config/config.toml

CONFIG_TOML=$HOME/.c4e-chain/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json

sudo systemctl restart c4ed
sudo journalctl -u c4ed -f -o cat
```

## Genesis

```bash
wget -O $HOME/.c4e-chain/config/genesis.json "https://raw.githubusercontent.com/chain4energy/c4e-chains/main/perun-1/genesis.json"
```
