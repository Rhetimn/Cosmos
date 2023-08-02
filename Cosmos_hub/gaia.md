# GAIA
## Packages
```bash
sudo apt update && sudo apt upgrade -y
```
## Dependencies
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```
## Go
```bash
ver="1.19.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
## Moniker, whrite something cool
```bash
NODENAME=Do_not_copypaste
```
## Make your custom ports. You can chose from 10 to 65
```bash
GAIA_PORT=35
```
## Save and import variables

```bash
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export GAIA_CHAIN_ID=cosmoshub-4" >> $HOME/.bash_profile
echo "export GAIA_PORT=${GAIA_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binaries

```bash
cd $HOME
wget https://github.com/cosmos/gaia/releases/download/v4.2.1/gaiad-v4.2.1-linux-amd64
mv gaiad-v4.2.1-linux-amd64 /usr/local/bin/gaiad
cd /usr/local/bin/
chmod +x gaiad
```
```bash
cd $HOME
wget https://github.com/cosmos/gaia/releases/download/v5.0.8/gaiad-v5.0.8-linux-amd64
mv gaiad-v5.0.8-linux-amd64 /usr/local/bin/gaiad
cd /usr/local/bin/
chmod +x gaiad
```

```bash
cd $HOME
wget https://github.com/cosmos/gaia/releases/download/v6.0.4/gaiad-v6.0.4-linux-amd64
mv gaiad-v6.0.4-linux-amd64 /usr/local/bin/gaiad
cd /usr/local/bin/
chmod +x gaiad
```

```bash
cd $HOME
wget https://github.com/cosmos/gaia/releases/download/v9.0.2/gaiad-v9.0.2-linux-amd64
mv gaiad-v9.0.2-linux-amd64 /usr/local/bin/gaiad
cd /usr/local/bin/
chmod +x gaiad
```


## Config app

```bash
gaiad config chain-id $GAIA_CHAIN_ID
gaiad config node tcp://localhost:${GAIA_PORT}657
```
## For your own risk 
```bash
gaiad config keyring-backend file
```

## Init app

```bash
gaiad init $NODENAME --chain-id $GAIA_CHAIN_ID
```
## Seeds and peers

```bash
SEEDS=""
PEERS="https://state-sync-01.theta-testnet.polypore.xyz,https://state-sync-02.theta-testnet.polypore.xyz"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.gaia/config/config.toml
```
## Genesis

```bash
cd $HOME
wget -O genesis.json https://snapshots.polkachu.com/genesis/cosmos/genesis.json --inet4-only
mv genesis.json ~/.gaia/config
```

## AddrBook
```bash
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/cosmos/addrbook.json --inet4-only
mv addrbook.json ~/.gaia/config
```

## Custom ports 

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${GAIA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${GAIA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${GAIA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${GAIA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${GAIA_PORT}660\"%" $HOME/.gaia/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${GAIA_PORT}317\"%; s%^address = \":8080\"%address = \":${GAIA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${GAIA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${GAIA_PORT}091\"%" $HOME/.gaia/config/app.toml
```
## Pruning

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gaia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gaia/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gaia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gaia/config/app.toml
```
## Minimum gas price

```bash
sed -E -i 's/minimum-gas-prices = \".*\"/minimum-gas-prices = \"0.00uatom\"/' $HOME/.gaia/config/app.toml
```

## Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gaia/config/config.toml
```
## Reset chain data

```bash
gaiad unsafe-reset-all --home $HOME/.gaia 
```
# Service

```bash
sudo tee /etc/systemd/system/gaiad.service > /dev/null <<EOF
[Unit]
Description=Stake to Kolot
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gaiad) start --home $HOME/.gaia --x-crisis-skip-assert-invariants
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```



## Register and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable gaiad
sudo systemctl restart gaiad && sudo journalctl -u gaiad -f -o cat
```
