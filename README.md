**Clone project repository**
```
cd && rm -rf node
git clone https://github.com/AssetMantle/node.git
cd node
git checkout v1.0.0
```

**Build binary**
```
make install
```

**Set node CLI configuration**
```
mantleNode config chain-id mantle-1
mantleNode config keyring-backend file
mantleNode config node tcp://localhost:14657
```

**Initialize the node**
```
mantleNode init "Your Node Name" --chain-id mantle-1
```
**Download genesis and addrbook files**
```
curl -L https://snapshots.nodejumper.io/assetmantle/genesis.json > $HOME/.mantleNode/config/genesis.json
curl -L https://snapshots.nodejumper.io/assetmantle/addrbook.json > $HOME/.mantleNode/config/addrbook.json
```

**Set seeds**
```
sed -i -e 's|^seeds *=.*|seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:14656,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@seeds.whispernode.com:14656,df949a46ae6529ae1e09b034b49716468d5cc7e9@seeds.stakerhouse.com:10156,4654c8bed4349e4800238cff1f88e97c1f880080@207.244.245.125:26656,a7aafd3330e57d3104be5b2820b6ad2d52ac19ec@3.39.94.36:26656,9c97f6143d3fae032af5f155d472bbc52f4d90b3@194.34.232.225:26656,fd096224f9c918089410ac7ab6d42d21ec87db60@65.21.230.230:26656,f33b2125c3b3a7c4838e22a060e38d2cefd66e78@65.108.140.109:26656,6261de9dac635a8fd8d19a70afc41f845c59db96@116.203.35.46:26656"|' $HOME/.mantleNode/config/config.toml
```

**Set minimum gas price**
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01umntl"|' $HOME/.mantleNode/config/app.toml
```

**Set pruning**
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.mantleNode/config/app.toml
```

**Change ports**
```
sed -i -e "s%:1317%:14617%; s%:8080%:14680%; s%:9090%:14690%; s%:9091%:14691%; s%:8545%:14645%; s%:8546%:14646%; s%:6065%:14665%" $HOME/.mantleNode/config/app.toml
sed -i -e "s%:26658%:14658%; s%:26657%:14657%; s%:6060%:14660%; s%:26656%:14656%; s%:26660%:14661%" $HOME/.mantleNode/config/config.toml
```

**Download latest chain data snapshot**
```
curl "https://snapshots.nodejumper.io/assetmantle/assetmantle_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.mantleNode"
```

**Create a service**
```
sudo tee /etc/systemd/system/mantleNode.service > /dev/null << EOF
[Unit]
Description=AssetMantle node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which mantleNode) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable mantleNode.service
```

# Start the service and check the logs
sudo systemctl start mantleNode.service
sudo journalctl -u mantleNode.service -f --no-hostname -o cat
