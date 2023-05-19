---
cover: ../../.gitbook/assets/ojo banner.jpeg
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
sudo systemctl stop ojod
cp $HOME/.ojo/data/priv_validator_state.json $HOME/.ojo/priv_validator_state.json.backup
rm -rf $HOME/.ojo/data

URL="https://snapshots-testnet.stake-town.com/ojo/ojo-devnet_latest.tar.lz4"
curl -L $URL | lz4 -dc - | tar -xf - -C $HOME/.ojo

mv $HOME/.ojo/priv_validator_state.json.backup $HOME/.ojo/data/priv_validator_state.json

sudo systemctl restart ojod && sudo journalctl -u ojod -f -o cat
```

## **State Sync**

```bash
sudo systemctl stop ojod

cp $HOME/.ojo/data/priv_validator_state.json $HOME/.ojo/priv_validator_state.json.backup
ojod tendermint unsafe-reset-all --home $HOME/.ojo --keep-addr-book

SNAP_RPC="https://ojo-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="18300f0a5973798c3900fe51ff255bb6bca982f9@65.109.65.248:36656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.ojo/config/config.toml

CONFIG_TOML=$HOME/.ojo/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.ojo/priv_validator_state.json.backup $HOME/.ojo/data/priv_validator_state.json

sudo systemctl restart ojod
sudo journalctl -u ojod -f -o cat
```

## **Address Book**

```bash
curl -Ls https://snapshots-testnet.stake-town.com/ojo/addrbook.json > $HOME/.ojo/config/addrbook.json
```

## Genesis

```bash
curl -Ls https://snapshots-testnet.stake-town.com/ojo/genesis.json > $HOME/.ojo/config/genesis.json
```
