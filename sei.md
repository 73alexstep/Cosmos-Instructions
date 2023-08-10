## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<YOUR_MONIKER_NAME_GOES_HERE>
```

Save and import variables into system
```
SEI_PORT=12
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export SEI_CHAIN_ID=atlantic-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

## Install go
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
git clone https://github.com/sei-protocol/sei-chain.git && cd sei-chain
git checkout 1.0.6beta
make install 
```

## Config app
```
seid config chain-id $SEI_CHAIN_ID
seid config keyring-backend test

```

## Init app
```
seid init $NODENAME --chain-id $SEI_CHAIN_ID
```

## Download genesis and addrbook
```
wget -qO $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-incentivized-testnet/genesis.json"
wget -qO $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-incentivized-testnet/addrbook.json"
```

## Set seeds and peers
```
SEEDS="df1f6617ff5acdc85d9daa890300a57a9d956e5e@sei-atlantic-1.seed.rhinostake.com:16660"
PEERS="22991efaa49dbaae857669d44cb564406a244811@18.222.18.162:26656,a37d65086e78865929ccb7388146fb93664223f7@18.144.13.149:26656,873a358b46b07c0c7c0280397a5ad27954a10633@141.95.175.196:26656,e66f9a9cab4428bfa3a7f32abbedbc684e734a48@185.193.17.129:12656,16225e262a0d38fe73073ab199f583e4a607e471@135.181.59.162:19656,2efd524f097b3fef2d26d0031fda21a72a51a765@38.242.213.174:12656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.sei/config/config.toml
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sei/config/config.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml
```

## Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usei\"/" $HOME/.sei/config/app.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.sei/config/config.toml
```

## Reset chain data
```
seid tendermint unsafe-reset-all --home $HOME/.sei
```

## Create service
```
sudo tee /etc/systemd/system/seid.service > /dev/null <<EOF
[Unit]
Description=sei
After=network-online.target

[Service]
User=$USER
ExecStart=$(which seid) start --home $HOME/.sei
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable seid
sudo systemctl restart seid && sudo journalctl -u seid -f -o cat
```
