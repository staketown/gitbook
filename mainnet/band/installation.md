---
cover: ../../.gitbook/assets/band-banner.jpeg
coverY: 0
---

# Installation

Install with one line script

```bash
bash <(curl -s https://raw.githubusercontent.com/staketown/cosmos/master/band/main_install.sh)
```

## Wallet creation

Create wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
bandd keys add <YOUR_WALLET_NAME>
```

Recover wallet

> ⚠️  store **seed** phrase, important during recovering

```bash
bandd keys add <YOUR_WALLET_NAME> --recover
```

## Validator creation

After successful synchronisation we can proceed with validation creation.

```bash
bandd tx staking create-validator \
--amount=1000000uband \
--pubkey=$(bandd tendermint show-validator) \
--moniker="<Your moniker>" \
--identity=<your identity> \
--details="<Your details>" \
--chain-id=laozi-mainnet \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=<YOUR_WALLET> \
--fees=5000uband
-y
```

## Set up Yoda <a href="#set-up-yoda" id="set-up-yoda"></a>

Yoda requires indexer to run properly. Please make sure if your node has set indexer in config.toml file as "kv". Before setting up Yoda, the Lambda function executor need to be set up to execute data sources. If this step has not been done yet, please follow the instructions on the following pages (select either one of these methods):

* [AWS Lambda Function](https://github.com/bandprotocol/data-source-runtime/wiki/Setup-Yoda-Executor-Using-AWS-Lambda) \\
* [Google Cloud Function](https://github.com/bandprotocol/data-source-runtime/wiki/Setup-Yoda-Executor-Using-Google-Cloud-Function)

To check Yoda version, use the following command.

```bash
yoda version
```

### Configure Yoda

Use the command below to config your Yoda.

```bash
rm -rf $HOME/.yoda
yoda config chain-id laozi-mainnet
yoda config node http://localhost:26657
yoda config broadcast-timeout "5m"
yoda config rpc-poll-interval "1s"
yoda config max-try 5
yoda config validator $(bandd keys show <YOUR_WALLET> -a --bech val)
```

Then, add multiple reporter accounts to allow Yoda to submit transactions concurrently.

```bash
yoda keys add REPORTER_1
yoda keys add REPORTER_2
yoda keys add REPORTER_3
yoda keys add REPORTER_4
yoda keys add REPORTER_5
```

:red\_circle:Make sure to save mnemonic phrases for all keys generated above

Lastly, configure the Lambda Executor endpoint to helps running data source scripts and return results to Yoda. More details about the executor can be found in [this section](https://docs.bandchain.org/develop/developer-guides/remote-data-source-executor).

```bash
export EXECUTOR_URL=<YOUR_EXECUTOR_URL>
yoda config executor "rest:${EXECUTOR_URL}?timeout=10s"
```

### Start Yoda <a href="#3-start-yoda" id="3-start-yoda"></a>

```bash
sudo tee /etc/systemd/system/yoda.service > /dev/null << EOF
[Unit]
Description=Yoda Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which yoda) run
Restart=always
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

The first time running Yoda, you will need to register and start yoda services by running the following commands.

```bash
sudo systemctl enable yoda
sudo systemctl start yoda
```

After yoda service has been started, logs can be queried by running `journalctl -u yoda.service -f` command. The log should be similar to the following log example below. Once verified, you can stop tailing the log by typing `Control-C`.

```
... systemd[...]: Started Yoda Daemon.
... yoda[...]: I[...] ⭐  Creating HTTP client with node URI: tcp://localhost:16957
... yoda[...]: I[...] 🚀  Starting WebSocket subscriber
... yoda[...]: I[...] 👂  Subscribing to events with query: tm.event = 'Tx'...
```

### Wait for the latest blocks to be synced <a href="#4-wait-for-the-latest-blocks-to-be-synced" id="4-wait-for-the-latest-blocks-to-be-synced"></a>

It is imperative to exercise caution and allow adequate time for the newly started BandChain node to synchronize its blocks until it has reached the latest block. The latest block can be verified on [CosmoScan](https://www.cosmoscan.io/blocks).

## Register Reporters and Become Oracle Provider <a href="#register-reporters-and-become-oracle-provider" id="register-reporters-and-become-oracle-provider"></a>

Yoda contains multiple reporters. You will need to register the reporters to help the validator submit transactions of reporting data.

Firstly, reporter accounts must be created on BandChain by supplying a small amount of BAND tokens.

```bash
bandd tx multi-send 1uband $(yoda keys list -a) \
  --from wallet \
  --chain-id laozi-mainnet
```

Secondly, register reporters to the validator, so that oracle requests for validator can be assigned to the reporters.

```
bandd tx oracle add-reporters $(yoda keys list -a) \
  --from wallet \
  --chain-id laozi-mainnet
```

Finally, activate the validator to become an oracle provider

```
bandd tx oracle activate \
  --from wallet \
  --chain-id laozi-mainnet
```

If all procedures are successful, then the oracle provider status for the validator should be active.

```
bandd query oracle validator $(bandd keys show -a wallet --bech val)

# {
#   "is_active": true,
#   "since": ...
# }
```

Congratulations, now you have become a validator on BandChain. Thanks #**kjnodes** for yoda instructions.
