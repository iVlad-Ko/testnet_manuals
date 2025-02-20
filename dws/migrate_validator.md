<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/166676803-ee125d04-dfe2-4c92-8f0c-8af357aad691.png">
</p>

# Migrate your validator to another machine

### 1. Run a new full node on a new machine
To setup full node you can follow my guide [Defund node setup for Testnet — deweb-testnet-1](https://github.com/kj89/testnet_manuals/blob/main/defund/README.md)

### 2. Confirm that you have the recovery seed phrase information for the active key running on the old machine

#### To backup your key
```
dewebd keys export mykey
```
> _This prints the private key that you can then paste into the file `mykey.backup`_

#### To get list of keys
```
dewebd keys list
```

### 3. Recover the active key of the old machine on the new machine

#### This can be done with the mnemonics
```
dewebd keys add mykey --recover
```

#### Or with the backup file `mykey.backup` from the previous step
```
dewebd keys import mykey mykey.backup
```

### 4. Wait for the new full node on the new machine to finish catching-up

#### To check synchronization status
```
curl -s localhost:26657/status | jq .result.sync_info
```
> _`catching_up` should be equal to `false`_

### 5. After the new node has caught-up, stop the validator node

> _To prevent double signing, you should stop the validator node before stopping the new full node to ensure the new node is at a greater block height than the validator node_
> _If the new node is behind the old validator node, then you may double-sign blocks_

#### Stop and disable service on old machine
```
sudo systemctl stop dewebd
sudo systemctl disable dewebd
```
> _The validator should start missing blocks at this point_

### 6. Stop service on new machine
```
sudo systemctl stop dewebd
```

### 7. Move the validator's private key from the old machine to the new machine
#### Private key is located in: `~/.deweb/config/priv_validator_key.json`

> _After being copied, the key `priv_validator_key.json` should then be removed from the old node's config directory to prevent double-signing if the node were to start back up_
```
mv ~/.deweb/config/priv_validator_key.json ~/.deweb/bak_priv_validator_key.json
```

### 8. Start service on a new validator node
```
sudo systemctl start dewebd
```
> _The new node should start signing blocks once caught-up_

### 9. Make sure your validator is not jailed
#### To unjail your validator
```
dewebd tx slashing unjail --chain-id deweb-testnet-1 --from mykey --gas=auto -y
```

### 10. After you ensure your validator is producing blocks and is healthy you can shut down old validator server
