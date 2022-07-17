# SEI
Official documentation: https://docs.seinetwork.io/nodes-and-validators/seinami-incentivized-testnet/joining-incentivized-testnet

Chain explorer: https://sei.explorers.guru/

## Minimum Hardware Requirements
3x CPUs; the faster clock speed the better
4GB RAM
80GB Disk
Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)
## Recommended Hardware Requirements
4x CPUs; the faster clock speed the better
8GB RAM
512GB of storage (SSD or NVME)
Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)
Filling out the first form after installing the validator: https://docs.google.com/forms/d/e/1FAIpQLSfD-FWT3VrxtYAAmUiwwX5Zbw3mzkZoT6pV0ZAXYqu1yUNtEw/viewform

The form after the assignment: https://docs.google.com/forms/d/1qxpIL-ATe1HMX87w1P7BjMqpjXExlKyo1_btEJi00JM/

## Set up your SEI full node

###Updating repositories
`sudo apt update && sudo apt upgrade -y`
### Installing the necessary utilities
sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y

### Setting up Go WITH ONE COMMAND
```
ver="1.18.1" && \ 
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \ 
sudo rm -rf /usr/local/go && \ 
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \ 
rm "go$ver.linux-amd64.tar.gz" && \ 
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \ 
source $HOME/.bash_profile && \ 
go version
```
### Installing the binary
```
git clone https://github.com/sei-protocol/sei-chain.git && cd sei-chain
git checkout 1.0.6beta
make install
```
`seid version --long | head`
### version 1.0.6beta 
### commit: e3958ff9cc3fa00a12b0c32cf55b635baa0d49bd

### Initialize a node to create the necessary configuration files
`seid init <name_moniker> --chain-id atlantic-1`

### Download Genesis
`wget -O $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json"`

### Download the addrbook
`wget -O $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/addrbook.json"`

### additional setup from the team
```
wget -qO optimize-configs.sh https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/optimize-configs.sh
sudo chmod +x optimize-configs.sh && ./optimize-configs.sh 
sudo systemctl restart seid && sudo journalctl -u seid -f -o cat
```
### (OPTIONAL) Configure pruning with one command app.toml
```
pruning="custom" && \ 
pruning_keep_recent="100" && \ 
pruning_keep_every="0" && \ pruning_interval="50" && \ 
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml && \ 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml && \ 
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml && \ 
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml
```
### Create a service file
```
sudo tee /etc/systemd/system/seid.service > /dev/null <<EOF 
[Unit] 
Description=seid 
After=network-online.target 

[Service] 
User=$USER 
ExecStart=$(which seid) 
start Restart=on-failure 
RestartSec=3 
LimitNOFILE=65535 

[Install] 
WantedBy=multi-user.target 
EOF
```
sudo systemctl daemon-reload && \ 
sudo systemctl enable seid && \ 
sudo systemctl restart seid && sudo journalctl -u seid -f -o cat

### Waiting for synchronization:
`seid status 2>&1 | jq .SyncInfo`

### Create or restore a wallet and save the output:
# create a wallet
`seid keys add <name_wallet>`
# regenerate the wallet (insert seed after the command)
`seid keys add <name_wallet> --recover`
## Don't forget to save seed !!!
### Creating a validator
```
seid tx staking create-validator \ 
--chain-id atlantic-1 \ 
--commission-rate 0.05 \ 
--commission-max-rate 0.2 \ 
--commission-max-change-rate 0.1 \ 
--min-self-delegation 1 \ 
--amount 1000000usei \ 
--pubkey $(seid tendermint show-validator) \ 
--moniker "<name_moniker>" \ 
--from <name_wallet> \ --fees 5550usei
```
###Create a validator
####Before creating a validator, please make sure you have at least 1 sei (1 sei equals 1000000 usei) and your node is synchronized

###To check your wallet balance:

`seid query bank balances $SEI_WALLET_ADDRESS`
### Don't forget to save priv_validator_key.json !!!
### After installation

### After completing the installation, please load the variables into the system

`source $HOME/.bash_profile`

Then you must make sure that your validator synchronizes the blocks. You can use the command below to check the synchronization status

seid status 2>&1 | jq .SyncInfo
Save Wallet Information
SEI_WALLET_ADDRESS=$(seid keys show $WALLET -a)
SEI_VALOPER_ADDRESS=$(seid keys show $WALLET --bech val -a)
echo 'export SEI_WALLET_ADDRESS='${SEI_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export SEI_VALOPER_ADDRESS='${SEI_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
To top up your wallet, go to the server Sei discord and go to the channel #atlantic-1-faucet

If your wallet shows no balance, your node is probably still syncing. Please wait until it finishes syncing and then continue
Useful Commands

Service management
Checking logs

journalctl -fu seid -o cat
Start the service

sudo systemctl start seid

Stop the service

sudo systemctl stop seid

Restart the service

sudo systemctl restart seid
To add a logo to mintscan:

https://github.com/cosmostation/cosmostation_token_resource fork
in the folder Moniker find the name of the project
via add file/upload file add your avatar. the file name must be valoper.png . and only png
PR
# collect commissions + rewards
seid tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 5555usei --commission -y
#delegate more to the steak (this is how 1 coin is sent)
seid tx staking delegate <valoper_address> 1000000usei --from <name_wallet> --fees 5555usei -y
# redeleting to another validator
seid tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000usei --from <name_wallet> --fees 5555usei -y
# unbond 
seid tx staking unbond <addr_valoper> 1000000usei --from <name_wallet> --fees 5555usei -y
# send coins to another address
seid tx bank send <name_wallet> <address> 1000000usei --fees 5555usei -y
# get out of jail
seid tx slashing unjail --from <name_wallet> --fees 5555usei -y
Also, if you have any questions, go to the discord and chat: https://discord.gg/brJfyazV

Telegram: https://t.me/seinetwork

GitHub: https://github.com/sei-protocol 

Gitbook: https://docs.seinetwork.io/introduction/overview 

Block Explorer: https://sei.explorers.guru/ 

Medium: https://medium.com/@seinetwork
