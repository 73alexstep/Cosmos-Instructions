## Links
- Explorer https://quicksilver.explorers.guru/
- Github https://github.com/ingenuity-build/testnets
- Website https://ingenuity.build/
- Twitter https://twitter.com/quicksilverzone

## Installation
Update server and install packages
```
sudo apt update && sudo apt upgrade -y && \
sudo apt-get install make git jq curl gcc g++ mc nano -y
```
Install GO
```
wget -O go1.18.linux-amd64.tar.gz https://go.dev/dl/go1.18.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.linux-amd64.tar.gz && rm go1.18.linux-amd64.tar.gz
cat <<'EOF' >> $HOME/.bash_profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
. $HOME/.bash_profile
cp /usr/local/go/bin/go /usr/bin
```
Check Go version
```
go version
```
### Install Quicksilver
```
cd $HOME
git clone https://github.com/ingenuity-build/quicksilver.git --branch v1.2.14
cd quicksilver
git checkout v1.2.14
make build
cp $HOME/quicksilver/build/quicksilverd /usr/local/bin
```
Check version
```
quicksilverd version
```
### Variables
Type YOURS and after that - copy it and past in command line interface on the server
```bash
#Set temporary variables
NODE_MONIKER=<Your_Moniker> ; \
YOUR_TEST_WALLET=<Your_Wallet_Name> ; \
CHAIN_ID=quicktest-3

#Check
echo $NODE_MONIKER ; \
echo $YOUR_TEST_WALLET ; \
echo $CHAIN_ID

#export
echo 'export NODE_MONIKER='\"${NODE_MONIKER}\" >> $HOME/.bash_profile ; \
echo 'export YOUR_TEST_WALLET='\"${YOUR_TEST_WALLET}\" >> $HOME/.bash_profile ; \
echo 'export chainName='\"${CHAIN_ID}\" >> $HOME/.bash_profile
```
### Init and perrs/seeds/genesis/and other ganges
```sh
quicksilverd init $NODE_MONIKER --chain-id $chainName
SEEDS="dd3460ec11f78b4a7c4336f22a356fe00805ab64@seed.quicktest-1.quicksilver.zone:26656" ; \
sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $HOME/.quicksilverd/config/config.toml

GENESIS_URL="https://raw.githubusercontent.com/ingenuity-build/testnets/main/rhapsody/genesis.json" ; \
NODE_HOME=$HOME/.quicksilverd ; \
curl -sSL $GENESIS_URL > $NODE_HOME/config/genesis.json

sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uqck\"/" $HOME/.quicksilverd/config/app.toml
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uqck\"/" $HOME/.quicksilverd/config/app.toml
sed -i.bak 's/^log_level/# log_level/' $HOME/.quicksilverd/config/config.toml
```
### Сreate a service file
```
sudo tee <<EOF >/dev/null /etc/systemd/system/quicksilverd.service
[Unit]
Description=Quicksilver Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which quicksilverd) start --log_level=info
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### Start
```
sudo systemctl enable quicksilverd
sudo systemctl daemon-reload
sudo systemctl restart quicksilverd
sudo systemctl status quicksilverd
```



