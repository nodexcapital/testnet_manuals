<p align="center">
  <img width="200" height="auto" src="https://user-images.githubusercontent.com/50621007/164164767-0a9590e5-b018-44de-8a3e-4ebdd905dfbc.png">
</p>

# Archway node setup for Incentivized Testnet — Torii-1
All information about testnet incentives and challenges you can find [here](https://philabs.notion.site/philabs/Archway-Incentivized-Testnet-Torii-1-9e70a8f431c041618c6932e70d46ccdd)

## Usefull tools I have created for Archway
> To generate gentx for torii-1 testnet please navigate to [Generate gentx for torii-1 incentivized testnet](https://github.com/kj89/testnet_manuals/blob/main/archway/gentx/README.md)
>
> To set up monitoring for your validator node navigate to [How to set up monitoring stack for your cosmos validator](https://github.com/kj89/cosmos_node_monitoring/blob/master/README.md)

## Set up your Archway fullnode
You can setup your Archway fullnode in few minutes by using automated script below. It will prompt you to input your validator node name!
```
wget -O archway.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/archway/archway.sh && chmod +x archway.sh && ./archway.sh
```

### Post installation
After installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
archwayd status 2>&1 | jq .SyncInfo
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
archwayd keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
archwayd keys add $WALLET --recover
```

To get current list of wallets
```
archwayd keys list
```

### Save wallet info
Add wallet address
```
WALLET_ADDRESS=$(archwayd keys show $WALLET -a)
echo 'export WALLET_ADDRESS='${WALLET_ADDRESS} >> $HOME/.bash_profile
```

Add valoper address
```
VALOPER_ADDRESS=$(archwayd keys show $WALLET --bech val -a)
echo 'export VALOPER_ADDRESS='${VALOPER_ADDRESS} >> $HOME/.bash_profile
```

Load variables into system
```
source $HOME/.bash_profile
```

### Top up your wallet balance using faucet
Currently there are two options of getting tokens in torii-1 testnet

1. Get 3 torii every 6 hours using Discord faucet
* connect to [Archway official Discord server](https://discord.gg/RVWwavhZ)
* choose your roles in `#get-roles` channel
* navigate to `#faucet` channel under `VALIDATORS` category
* input faucet command with your wallet address. Here is an example, `!faucet archway1yh9xhsnnveys532df2nfsuwj44pj6kf4h2thd6`
> If in response you get faucet failed than your wallet reached the limit and you will not able to request more tokens

2. Get 50 torii using your twitter account
* navigate to https://stakely.io/en/faucet/archway-testnet
* input your wallet address and click verify
* then press tweet button and wait for faucet to process your tweet
> Please note: to mitigate against spam accounts, the Stakely faucet will discard requests from users with Twitter accounts with very little activity or that are too new!

### Create validator
Before creating validator please make sure that you have at least 1 torii (1 torii is equal to 1000000 utorii)

To check your wallet balance:
```
archwayd query bank balances $WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
archwayd tx staking create-validator \
  --amount 1000000utorii \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(archwayd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $CHAIN_ID
```

## Security
To protect you keys please make sure you at least follow basic security

### Set up ssh keys for authentication
Good tutorial on how to set up ssh keys for authentication to your server can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security
Start by checking the status of ufw.
```
sudo ufw status
```

Sets the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow 26656
sudo ufw enable
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu archwayd -o cat
```

Start service
```
systemctl start archwayd
```

Stop service
```
systemctl stop archwayd
```

Restart service
```
systemctl restart archwayd
```

### Node info
Synchronization info
```
archwayd status 2>&1 | jq .SyncInfo
```

Validator info
```
archwayd status 2>&1 | jq .ValidatorInfo
```

Node info
```
archwayd status 2>&1 | jq .NodeInfo
```

Show node id
```
archwayd tendermint show-node-id
```

### Wallet operations
List of wallets
```
archwayd keys list
```

Recover wallet
```
archwayd keys add $WALLET --recover
```

Delete wallet
```
archwayd keys delete $WALLET
```

Get wallet balance
```
archwayd query bank balances $WALLET_ADDRESS
```

Transfer funds
```
archwayd tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000utorii
```

### Staking, Delegation and Rewards
Delegate stake
```
archwayd tx staking delegate $VALOPER_ADDRESS 10000000utorii --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
archwayd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000utorii --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Withdraw all rewards
```
archwayd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
archwayd tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CHAIN_ID
```

### Validator management
Edit validator
```
archwayd tx staking edit-validator \
--moniker=$NODENAME \
--identity=1C5ACD2EEF363C3A \
--website="http://kjnodes.com" \
--details="Providing professional staking services with high performance and availability. Find me at Discord: kjnodes#8455 and Telegram: @kjnodes" \
--chain-id=$CHAIN_ID \
--from=$WALLET
```

Unjail validator
```
archwayd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CHAIN_ID \
  --gas=auto
```