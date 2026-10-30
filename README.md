<img width="1024" height="572" alt="image" src="https://github.com/user-attachments/assets/a3a7d146-a210-4733-9304-50d0ff9bf46d" />

* Celestia Mocha test ağı uzun süredir aktif bir şekilde çalışıyor ve geliştirmeler sürekli devam ettiği için belirsiz bir süre zarfında çalışmaya devam edecek. Ağ şu an mocha-5 üzerinde çalışıyor (mocha-4'teki validatorler taşınmadı, mocha-5'te yeniden oluşturulması gerekiyor).

* Bu kılavuz içerisinde adım adım ilerleyerek test ağında Validator, Fibre, Bridge ve Light Node çalıştırabilir, ağa katkıda bulunabilirsiniz. Kılavuz güncel sürümlere göre hazırlanmıştır (celestia-app v10.5.0-mocha, celestia-node v0.34.4-mocha).

* Full Storage Node tipi celestia-node'dan kaldırıldı. Bütün veriyi tutmak isteyenler Bridge Node'u --archival ile çalıştırabilir.

* Kurulum sırasında veya sonrasında bir şey sormak isterseniz bana Telegram, Mail ve Discord yoluyla ulaşabilirsiniz. Ayrıca GitHub üzerinden de katkı sağlayabilirsiniz.

## Validator Node Minimum Sistem Gereksinimleri

 - Memory: 32 GB RAM
 - CPU: 32 Cores (GFNI ve SHA-NI destekli, CPU testinden geçmeli)
 - Disk: 2 TB NVMe SSD Storage
 - Bandwidth: 1 Gbps
 - İşletim Sistemi: Ubuntu 24.04 (celestia-app glibc 2.38 istiyor, Ubuntu 22.04'te çalışmaz)

NOT: Mocha artık mocha-5 ağında. mocha-4'teki validatorler mocha-5'e taşınmadı, validatorünüzü mocha-5'te yeniden oluşturmanız gerekiyor.

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu lz4 aria2 -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.26.9"
cd $HOME
wget "https://go.dev/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### Dosyaları Yüklemek
```
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=v10.5.0-mocha
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
celestia-appd version
```

### Inıt Node
```
celestia-appd init "Node Name" --chain-id mocha-5
```

### Genesis İndirmek
```
celestia-appd download-genesis mocha-5
```

### Seeds ve Peers Eklemek
```
SEEDS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/main/mocha-5/seeds.txt | tr '\n' ',' | sed 's/,$//')
PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/main/mocha-5/peers.txt | tr '\n' ',' | sed 's/,$//')
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

### Fibre İçin gRPC ve İmzalayıcıyı Açmak (Validatorler İçin Gerekli)
- Fibre sunucusu node'a bu iki adresten bağlanır. İkisi de sadece 127.0.0.1'de dinler, dışarıya açılmaz.
```
sed -i -e '/^\[grpc\]/,/^\[/{s/^enable *=.*/enable = true/; s/^address *=.*/address = "127.0.0.1:9090"/}' $HOME/.celestia-app/config/app.toml
sed -i -e 's/^priv_validator_grpc_laddr *=.*/priv_validator_grpc_laddr = "127.0.0.1:26669"/' $HOME/.celestia-app/config/config.toml
```

### Cüzdan Oluşturnmak
- Aşağıdaki kodu girdiğinizde ekranda size şifre koymanızı ister ve bundan sonra cüzdan adresinizi, cüzdanın gizli kelimelerini gösteriri. Bunları mutlaka kaydedin, herhangi bir geri getirme durumunda bu kelimeler gerekli.

```
celestia-appd keys add cuzdanadı
```

### Test Tokenı almak
- https://discord.com/invite/YsnTPcSfWQ discord kanalına giderek #mocha-faucet kanalında `$request CUZDAN_ADRESINIZ` yazarak test tokenı alabilirsiniz. Haftada adres başına 10 token verilir.
- Discord kullanmak istemezseniz web faucet: https://mocha.celenium.io/faucet

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
celestia-appd status 2>&1 | jq .sync_info.catching_up
```

### Validator Oluşturmak
- Sisteme eşitlendikten sonra aşağıdaki kod ile validator oluşturabilirsiniz. 1 adet test tokenına göre ayarlanmıştır.
```
celestia-appd tx staking create-validator \
--amount 1000000utia \
--from cuzdanismi \
--moniker "Monikerismi" \
--chain-id mocha-5 \
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

NOT: Ağda olan bütün durumları ve tx sonuçarı için [Explorer](https://testnet.celestia.explorers.guru/) veya [Celenium](https://mocha.celenium.io) sitelerini ziyaret edebilirsiniz.


## Fibre Sunucusu Kurulumu (Validatorler İçin)

- Fibre, v10 ile gelen yeni veri erişilebilirliği protokolü. Aktif (bonded) bir validatorünüz olmalı.
- Şimdilik Fibre'ı validator ile aynı sunucuda çalıştırmanız öneriliyor. Fibre verisi ile celestia-app verisi için ayrı diskler kullanın.
- Yukarıdaki "Fibre İçin gRPC ve İmzalayıcıyı Açmak" adımını yapıp node'u yeniden başlatmış olmalısınız.

### Fibre Yüklemek
- Fibre, celestia-app ile aynı sürümden indirilir.
```
cd $HOME
wget https://github.com/celestiaorg/celestia-app/releases/download/v10.5.0-mocha/fibre_Linux_x86_64.tar.gz
wget https://github.com/celestiaorg/celestia-app/releases/download/v10.5.0-mocha/checksums.txt
sha256sum --ignore-missing --check checksums.txt
tar -xzf fibre_Linux_x86_64.tar.gz
sudo mv fibre /usr/local/bin/
rm fibre_Linux_x86_64.tar.gz checksums.txt
fibre version
```

### v10 Aktif mi Kontrol
- Sonuçta app_version 10 veya üstü yazmalı.
```
curl -s http://127.0.0.1:26657/abci_info | jq .result.response.app_version
```

### Servis Oluşturmak
- FIBRE_HOME klasörünü celestia-app'ten farklı bir diskte seçin. Aşağıda örnek olarak /data/fibre kullanılmıştır.
```
sudo mkdir -p /data/fibre
sudo tee /etc/systemd/system/fibre.service > /dev/null <<EOF
[Unit]
Description=Celestia Fibre server
After=network-online.target celestia-appd.service
[Service]
User=root
ExecStart=/usr/local/bin/fibre start --home /data/fibre --app-grpc-address 127.0.0.1:9090 --signer-grpc-address 127.0.0.1:26669 --server-listen-address 0.0.0.0:7980
Restart=on-failure
RestartSec=5
TimeoutStopSec=120
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Fibre Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable fibre
sudo systemctl restart fibre && sudo journalctl -u fibre -f
```

### Fibre Portunu Açmak
- 7980 portu dışarıdan erişilebilir olmalı. 9090 ve 26669 portlarını kesinlikle dışarı açmayın.
```
sudo ufw allow 7980/tcp
```

### Fibre Adresini Zincire Kaydetmek
- Fibre çalışıp dışarıdan erişilebilir olduktan sonra yapın. SUNUCU_IP yerine sunucunuzun IP'sini veya alan adını yazın.
```
celestia-appd tx valaddr set-host SUNUCU_IP:7980 \
--from cuzdanismi \
--chain-id mocha-5 \
--gas 200000 \
--fees 2000utia \
-y
```

### Kayıt Kontrolü
- Sonuçta found: true ve kendi adresiniz görünmeli.
```
celestia-appd query valaddr provider $(celestia-appd comet show-address)
```

### Güncellemeden Sonra Fibre Ayarlarını Eşitlemek
- Her sürüm güncellemesinden sonra yeni ayar alanlarını server_config.toml dosyasına eklemek için çalıştırın.
```
celestia-appd config sync --fibre-home /data/fibre --dry-run
celestia-appd config sync --fibre-home /data/fibre
sudo systemctl restart fibre
```


## Bridge Node Minimum Sistem Gereksinimleri

 - Memory: 64 GB RAM
 - CPU: 32 Cores
 - Disk: 2 TB NVMe SSD Storage (Archival için 12 TB)
 - Bandwidth: 1 Gbps
 - İşletim Sistemi: Ubuntu 24.04

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.26.9"
cd $HOME
wget "https://go.dev/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### Celestia-Node Yüklemek
```
cd $HOME
rm -rf celestia-node
git clone https://github.com/celestiaorg/celestia-node.git
cd celestia-node/
git checkout tags/v0.34.4-mocha
make build
make install
make cel-key
```

- Versiyon Kontrol ```celestia version``` >>> v0.34.4-mocha

### Init İşlemi
- Kendi consensus node'unuz varsa rpc-mocha.pops.one yerine onun IP'sini yazın (o node'da gRPC açık olmalı).
```
celestia bridge init --core.ip rpc-mocha.pops.one --core.port 9090 --p2p.network mocha
```

### Cüzdan Oluşturma
```
./cel-key list --node.type bridge --p2p.network mocha --keyring-backend test
```

### Faucetten Token Almak
- Discord kanalına giderek cüzdan adresinize test token almalısınız >>> https://discord.com/invite/YsnTPcSfWQ (#mocha-faucet kanalında `$request CUZDAN_ADRESINIZ`)

### Servis Oluşturma
- Archival bridge istiyorsanız ExecStart satırının sonuna --archival ekleyin. Pruned başlatılan node sonradan archival'a çevrilemez.
```
sudo tee /etc/systemd/system/celestia-bridge.service > /dev/null <<EOF
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia bridge start --core.ip rpc-mocha.pops.one --core.port 9090 --keyring.backend test --keyring.keyname my_celes_key --p2p.network mocha
Restart=on-failure
RestartSec=3
LimitNOFILE=100000

[Install]
WantedBy=multi-user.target
EOF
```

### Portu Açmak
- Diğer node'ların sizi bulabilmesi için 2121 portu açık olmalı.
```
sudo ufw allow 2121
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
- Aynı sunucuda validator çalışıyorsa $HOME/.celestia-app klasörünü SİLMEYİN, o validator verisidir.
```
sudo systemctl stop celestia-bridge
sudo systemctl disable celestia-bridge
sudo rm /etc/systemd/system/celestia-bridge.service
rm -rf $HOME/celestia-node
rm -rf $HOME/.celestia-bridge-mocha-5
```

## Full Storage Node

- Full storage node tipi celestia-node'dan kaldırıldı. Artık sadece bridge ve light node var. Bütün veriyi tutmak istiyorsanız yukarıdaki bridge node kurulumunu --archival ile yapın.
- Eski full node'unuz varsa aşağıdaki kodlarla silebilirsiniz.
```
sudo systemctl stop celestia-full
sudo systemctl disable celestia-full
sudo rm /etc/systemd/system/celestia-full.service
rm -rf $HOME/.celestia-full-mocha-4
```

## Light Node Sistem Gereksinimleri

 - Memory: 500 MB RAM (Minimum)
 - CPU: Single Core
 - Disk: 20 GB SSD Storage
 - Bandwidth: 56 Kbps

### Sistem Güncellemeleri
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y
```

### Go Yüklemek
```
ver="1.26.9"
cd $HOME
wget "https://go.dev/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### Celestia-Node Yüklemek
```
cd $HOME
rm -rf celestia-node
git clone https://github.com/celestiaorg/celestia-node.git
cd celestia-node/
git checkout tags/v0.34.4-mocha
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
celestia light init --p2p.network mocha
```

### Servis Oluşturma
```
sudo tee /etc/systemd/system/celestia-light.service > /dev/null <<EOF
[Unit]
Description=celestia-light Cosmos daemon
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/celestia light start --core.ip rpc-mocha.pops.one --core.port 9090 --keyring.backend test --keyring.keyname my_celes_key --p2p.network mocha
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
sudo systemctl enable celestia-light
sudo systemctl start celestia-light && sudo journalctl -u celestia-light -f
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
sudo rm /etc/systemd/system/celestia-light.service
rm -rf $HOME/celestia-node
rm -rf $HOME/.celestia-light-mocha-5
```

### bbr Aktif Hale Getirmek (Mutlaka Yapın)
- Aşağıdaki kodları girerek basit bir şekilde aktifleştirebilirsiniz.
```
cd $HOME/celestia-app
make bbr-enable
make bbr-check
```

- Eğer yukarıdaki kodlarda hata alırsanız aşağıdaki kodla aktif hale getirebilirsiniz.
```
sudo modprobe tcp_bbr
echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
