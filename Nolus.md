## Update packeges
```bash
sudo apt update && sudo apt upgrade
```
### Install some tools
```bash
sudo apt install curl make clang ncdu bsdmainutils build-essential git jq pkg-config tmux htop mc -y
```
### Install go
```bash
cd $HOME
wget -O go1.19.2.linux-amd64.tar.gz https://golang.org/dl/go1.19.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz && rm go1.19.2.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```

### Install Nolus and Init
```bash
rm -rf $HOME/nolus-core
cd $HOME
git clone https://github.com/Nolus-Protocol/nolus-core
cd nolus-core
git checkout v0.2.1-testnet
make build
sudo mv ./target/release/nolusd /usr/local/bin/
nolusd init "$NOLUS_NODENAME" --chain-id=nolus-rila
```
### Seeds and Peers
```bash
#seeds="8e1590558d8fede2f8c9405b7ef550ff455ce842@51.79.30.9:26656,bfffaf3b2c38292bd0aa2a3efe59f210f49b5793@51.91.208.71:26656,106c6974096ca8224f20a85396155979dbd2fb09@198.244.141.176:26656"
peers="56cee116ac477689df3b4d86cea5e49cfb450dda@54.246.232.38:26656,56f14005119e17ffb4ef3091886e6f7efd375bfd@34.241.107.0:26656,7f26067679b4323496319fda007a279b52387d77@63.35.222.83:26656,7f4a1876560d807bb049b2e0d0aa4c60cc83aa0a@63.32.88.49:26656,3889ba7efc588b6ec6bdef55a7295f3dd559ebd7@3.249.209.26:26656,de7b54f988a5d086656dcb588f079eb7367f6033@34.244.137.169:26656"
#sed -i "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.nolus/config/config.toml
sed -i.default "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.nolus/config/config.toml
sed -i.default 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0025unls"/g' $HOME/.nolus/config/app.toml
```
### Pruning
```sh
sed -i "s/pruning *=.*/pruning = \"custom\"/g" $HOME/.nolus/config/app.toml
sed -i "s/pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/g" $HOME/.nolus/config/app.toml
sed -i "s/pruning-interval *=.*/pruning-interval = \"10\"/g" $HOME/.nolus/config/app.toml
```
### unsafe-reset-all
```bash
wget -O $HOME/.nolus/config/genesis.json https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/genesis.json
nolusd tendermint unsafe-reset-all --home $HOME/.nolus
```
### Create servicw file
```bash
echo "[Unit]
Description=Nolus Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/nolusd start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/nolusd.service
```
### Move servise
```bash
sudo mv $HOME/nolusd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```
### Start
```sh
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable nolusd
sudo systemctl restart nolusd
```
