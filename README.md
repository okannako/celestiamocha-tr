![207](https://github.com/okannako/celestiamocha/assets/73176377/55fab58c-cf7e-4062-ad3b-8f981612a648)

* Celestia Mocha test ağı uzun süredir aktif bir şekilde çalışıyor ve geliştirmeler sürekli devam ettiği için belirsiz bir süre zarfında çalışmaya devam edecek.

* Bu kılavuz içerisinde adım adım ilerleyerek test ağında Validator, Bridge, Full Storage ve Light Node çalıştırabilir, ağa katkıda bulunabilirsiniz.

* Kurulumn sırasında veya sonrasında bir şey sormak isterseniz bana Telegram, Mail ve Discord yoluyla ulaşabirsiniz. Ayrıca Github üzerinden de katkı sağlayabilirsiniz.

## Validator Node Minimum Sistem Gereksinimleri

 - Memory: 8 GB RAM
 - CPU: 6 Cores
 - Disk: 500 GB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.23.1"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Dosyaları Yüklemek
```
cd $HOME 
rm -rf celestia-app 
git clone https://github.com/celestiaorg/celestia-app.git 
cd celestia-app/ 
APP_VERSION=v4.0.6-mocha
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
celestia version
```

### Inıt Node
```
celestia-appd init "Node Name" --chain-id mocha-4
```

### Genesis ve Addrbook İndirmek
```
wget -O $HOME/.celestia-app/config/genesis.json https://mainnet-files.itrocket.net/celestia/genesis.json
wget -O $HOME/.celestia-app/config/addrbook.json https://mainnet-files.itrocket.net/celestia/addrbook.json
```

### Seeds ve Peers Eklemek
```
SEEDS="5d0bf034d6e6a8b5ee31a2f42f753f1107b3a00e@celestia-testnet-seed.itrocket.net:11656"
PEERS="daf2cecee2bd7f1b3bf94839f993f807c6b15fbf@celestia-testnet-peer.itrocket.net:11656,d468354f164a374f9560d6ad46572668020a222e@195.14.6.178:26656,6ed983017167d96c62b166725250940deb783563@65.108.142.147:27656,23711b72518bf5ce249d3f06110858cefc5f294a@94.130.54.216:11656,a98484ac9cb8235bd6a65cdf7648107e3d14dab4@116.202.231.58:12056,ac4df0b6796aed28a3fae0e95f7828c88a341da4@217.160.102.31:26656,5a0de83958f2895cdc6265a441898a54e52a485f@72.46.84.33:26656,3b1e36486b319ab99e7e12a9d56d8031a46e9139@15.235.65.137:26656,8194b4f9c4d558a0a4d4242bce9274892cbfb386@20.250.38.245:26656,ad64e0055d33445ce4b2f953b7910ae63987aeb2@148.113.8.171:33656,edebca7508b70df9659c1293b0d8cbc05c77c91f@65.108.12.253:16007,3e30bcfc55e7d351f18144aab4b0973e9e9bf987@65.108.226.183:11656,85aef6d15d0197baff696b6e31c88e0f21073c59@162.55.245.144:2400,f07813ee16dabdeb370c7ffbdbbc73d9f4db48d5@139.45.205.58:28656,2c0c7aaeac21af6f6cd4f3c561b1a5ea22e39460@62.138.24.120:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml
```

### Pruning Açmak (Zorunlu Değil)
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.celestia-app/config/app.toml
```

### Minimum Gas Price Eklemek
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.002utia"|g' $HOME/.celestia-app/config/app.toml
```

### Cüzdan Oluşturnmak
- Aşağıdaki kodu girdiğinizde ekranda size şifre koymanızı ister ve bundan sonra cüzdan adresinizi, cüzdanın gizli kelimelerini gösteriri. Bunları mutlaka kaydedin, herhangi bir geri getirme durumunda bu kelimeler gerekli.

```
celestia-appd keys add cuzdanadı
```

### Test Tokenı almak
- https://discord.gg/nWqg8BGAUd discord kanalına giderek Faucet kanalından yukarıdaki kodu girdiğinizde size verdiği cüzdana test tokenı almak zorundasınız.

### Servis Oluşturmak
```
sudo tee /etc/systemd/system/celestia-appd.service > /dev/null <<EOF
[Unit]
Description=Celestia node
After=network-online.target
[Service]
User=root
ExecStart=$(which celestia-appd) start --home $HOME/.celestia-app
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Node Başlatmnak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f
```

### Sisteme Eşitlenme Kontrolü
- Kodu girdiğinizde sonuç olarak false verdiğinde sisteme eşitlenmiş demektir.

```
celestia-appd status 2>&1 | jq 
```

### Validator Oluşturmak
- Sisteme eşitlendikten sonra aşağıdaki kod ile validator oluşturabilirsiniz. 1 adet test tokenına göre ayarlanmıştır.
```
celestia-appd tx staking create-validator \
--amount 1000000utia \
--from cuzdanismi \
--moniker "Monikerismi" \
--chain-id mocha \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(celestia-appd tendermint show-validator) \
--gas 500000 \
--fees 3000utia \
--identity "" \
--website "" \
--details "" \
-y
```

NOT: Ağda olan bütün durumları ve tx sonuçarı için [Explorer](https://testnet.celestia.explorers.guru/) için siteyi ziyaret edebilirsiniz.


## Bridge Node Minimum Sistem Gereksinimleri

 - Memory: 16 GB RAM (Minimum)
 - CPU: 6 Cores
 - Disk: 10 TB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.23.1"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Celestia-Node Yüklemek
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.23.1-mocha 
make build 
make install 
make cel-key 
```

- Versiyon Kontrol ```celestia version``` >>> v0.23.1-mocha

### Init İşlemi
```
celestia bridge init --core.ip rpc-mocha.pops.one:26657 --p2p.network mocha-4
```

### Cüzdan Oluşturma
```
./cel-key list --node.type bridge --p2p.network mocha --keyring-backend test
```

### Faucetten Token Almak
- Discord kanalına giderek cüzdan adresinize test token almalısınız >>> https://discord.gg/wpuN2xx7yQ

### Servis Oluşturma
```
sudo tee /etc/systemd/system/celestia-bridge.service > /dev/null <<EOF
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia bridge start --core.ip rpc-mocha.pops.one:26657 --core.rpc.port 26657 --core.grpc.port 9090 --keyring.backend test --keyring.keyname my_celes_key --metrics.tls=true --metrics --metrics.endpoint otel.celestia-mocha.com --p2p.network mocha
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

### Bridge Node Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-bridge
sudo systemctl start celestia-bridge && sudo journalctl -u celestia-bridge -f
```

### Node Id Öğrenmek
```
AUTH_TOKEN=$(celestia bridge auth admin --p2p.network mocha)
```
```
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

### Bridge Node Silmek
```
sudo systemctl stop celestia-bridge
sudo systemctl disable celestia-bridge
rm -rf $HOME/celestia-node  
rm -rf $HOME/.celestia-bridge-mocha-4
rm -rf $HOME/.celestia-app
```

## Full Storage Node Sistem Gereksinimleri

 - Memory: 16 GB RAM (Minimum)
 - CPU: Quad-Core Cores
 - Disk: 10 TB SSD Storage
 - Bandwidth: 1 Gbps for Download/100 Mbps for Upload

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.23.1"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Celestia-Node Yüklemek
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.23.1-mocha 
make build 
make install 
make cel-key 
```

### Init İşlemi
```
celestia full init --core.ip rpc-mocha.pops.one:26657 --p2p.network mocha
```

### Cüzdan Oluşturma
```
./cel-key list --p2p.network mocha --node.type full --keyring-backend test
```

### Servis Oluşturma
```
sudo tee /etc/systemd/system/celestia-full.service > /dev/null <<EOF
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia full start --core.ip rpc-mocha.pops.one:26657 --core.rpc.port 26657 --core.grpc.port 9090 --keyring.backend test --keyring.keyname my_celes_key --metrics.tls=true --metrics --metrics.endpoint otel.celestia-mocha.com --p2p.network mocha
Restart=on-failure
RestartSec=3
LimitNOFILE=100000
WantedBy=multi-user.target
EOF
```

### Node Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-full
sudo systemctl start celestia-full && sudo journalctl -u celestia-full -f
```

### Node Id Öğrenmek
```
AUTH_TOKEN=$(celestia full auth admin --p2p.network mocha)
```
```
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

### Full Storage Node Silmek
```
sudo systemctl stop celestia-full
sudo systemctl disable celestia-full
rm -rf $HOME/celestia-node  
rm -rf $HOME/.celestia-full-mocha-4
rm -rf $HOME/.celestia-app
```

## Light Node Sistem Gereksinimleri

 - Memory: 500 MB RAM (Minimum)
 - CPU: Single Core
 - Disk: 100 GB SSD Storage
 - Bandwidth: 56 Kbps 

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.23.1"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Celestia-Node Yüklemek
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.23.1-mocha 
make build 
make install 
make cel-key 
```

### Cüzdan Oluşturma
```
./cel-key list --node.type light --p2p.network mocha --keyring-backend test
```

### Init İşlemi
```
celestia light init --core.ip rpc-mocha.pops.one:26657 --p2p.network mocha
```

### Servis Oluşturma
```
sudo tee /etc/systemd/system/celestia-light.service > /dev/null <<EOF
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia light start --core.ip rpc-mocha.pops.one:26657 --core.rpc.port 26657 --core.grpc.port 9090 --keyring.backend test --keyring.keyname my_celes_key  --keyring.keyname my_celes_key --metrics.tls=true --metrics --metrics.endpoint otel.celestia-mocha.com --p2p.network mocha
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

### Node Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-full
sudo systemctl start celestia-full && sudo journalctl -u celestia-full -f
```

### Node Id Öğrenmek
```
AUTH_TOKEN=$(celestia light auth admin --p2p.network mocha)
```
```
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```

### Light Node Silmek
```
sudo systemctl stop celestia-light
sudo systemctl disable celestia-light
rm -rf $HOME/celestia-node  
rm -rf $HOME/.celestia-light-mocha
rm -rf $HOME/.celestia-app
```

### bbr Aktif Hale Getirmek (Mutlaka Yapın)
- Aşağıdaki kodları girerek basit bir şekilde aktifleştirebilirsiniz.
```
cd celestia-app
make enable-bbr
```

- Eğer yukarıdaki kodlarda hata alırsanız aşağıdaki kodla aktif hale getirebilirsiniz.
```
sudo modprobe tcp_bbr; \
        echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf; \
        echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.conf; \
        sudo sysctl -p; \
```
