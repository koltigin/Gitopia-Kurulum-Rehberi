# GitopiaTurkceKurulumRehberi
![image](https://user-images.githubusercontent.com/102043225/176181117-e8589025-8649-4ac3-9308-08c48f0335fe.png)

# Gitopia Türkçe Kurulum Rehberi
Kaynak: https://docs.gitopia.com/validator-setup/index.html
Explorer: https://explorer.gitopia.com/
## Sistem Gereksinimleri (Minimum) 
* 2CPU
* 4GB RAM

## Gerekli Kurulumlar
Burada gerekli olan ubuntu güncellemelerini ve kütüphaneleri yükleyeceğiz. Aşağıdaki kodları sırasıyla giriniz.
```
sudo apt update && sudo apt upgrade -y
sudo apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen gcc
sudo apt-get install -y make gcc
sudo apt autoremove
```

## Go' Kurulumu ve Ayarlanması

```
wget https://golang.org/dl/go1.18.linux-amd64.tar.gz

sudo tar -C /usr/local -xzf go1.18.linux-amd64.tar.gz

cat <<EOF >> ~/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source ~/.profile
go version
```

Son olarak aşağıdaki gibi bir çıktı aldıysanız, her şey yolundadır.
```
go version go1.18 linux/amd64
```

## Gitopia'nın Kuruluması

```
curl https://get.gitopia.com | bash

git clone -b v0.13.0 gitopia://gitopia1dlpc7ps63kj5v0kn5v8eq9sn2n8v8r5z9jmwff/gitopia

cd gitopia && make install
```

Yukarıdaki kodu girdikten sonra ekranızda durum aşağıdaki gibiyse sorun yoktur;
```
root@sunucuadi:~/gitopia#
```

Kuruluma devam ediyoruz.
```
git clone gitopia://gitopia1dlpc7ps63kj5v0kn5v8eq9sn2n8v8r5z9jmwff/testnets

cd
```

### Validator (Moniker) Adı Oluşturulması
Aşağıdaki kodda Koltigin yazan yere kendi validatör adınızı girin.
```
GITOPIA_MONIKER="Koltigin"

gitopiad init --chain-id "gitopia-janus-testnet" "$GITOPIA_MONIKER"
```

Yukarıdaki iki kodu girdikten sonra uzunca bir kod çıktısı göreceksiniz. Bu kodun sonunda aşağıdaki gibi Koltigin yazan yerde kendi moniker adınızı görüyorsanız problem yoktur.
```
............"","moniker":"Koltigin","node_id":"....................."}
root@sunucuadi:~#
```

### Genesis Dosyası Oluşturuyoruz
Kuruluma devam ediyoruz, genesis dosyasını oluşturuyoruz.
```
cp gitopia/testnets/gitopia-janus-testnet/genesis.json .gitopia/config/
gitopiad validate-genesis
```

### Servis Dosyası Oluşturuyoruz
```
nano /etc/systemd/system/gitopiad.service
```

Açılan sayfaya aşağıdaki kodu olduğu gibi kopyalayıp yapıştırıyoruz
```
[Unit]
Description=gitopia
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/gitopiad start
Restart=always
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
```

Dosyasyı kapatmak için önce CTRL+X ardından Y sonra da Enter'a basıp dosyayı kaydediyoruz.

### Screen Oluşturuyoruz ve Daha Sonra Node'u Başlatıyoruz

Öncelikle aşağıdaki kodu girerek screen yüklüyoruz;
```
sudo apt install screen
```

Kurulum tamamlandıktan sonra scrren oluşturuyoruz. Ben aşağıdaki kodda screen adını Gitopia yaptım. sizler isterseniz kendi adınızı da yazabilirsiniz.
```
screen -S Gitopia
```

Açılan boş screen'da aşağıdaki kodları girerek node'umuzu başlatıyoruz ve logları yüklüyoruz.
```
systemctl daemon-reload
systemctl start gitopiad
systemctl enable gitopiad
journalctl -u gitopiad -f
```

Loglar yüklenmeyecek çünkü seed bekleyeceğiz.

## Daha Sonra Yapılacaklar (Bu Bölüm Güncenllenecektir)

### Minimum GAS Ücretini Ayarlama 
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001utlore\"/" $HOME/.gitopia/config/app.toml
```

### SEEDS ve PEERS Ayarlama 

```
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.gitopia/config/config.toml
```
### Prometheus'u Aktif Etme 
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gitopia/config/config.toml

## Cüzdan

### Yeni Cüzdan Oluşturma
Eğer Yeni Kurulum yapıyorsanız buradan devam ediniz. Yeni cüzdanınız oluştuktan sonra size kurtarma kelimelerini verecektir. Kodun çıktsısını kopyalayıp kaydetmeyi unutmayın!
```
WALLET=<cüzdan-adınız>
gitopiad keys add "$WALLET"
```

### Var Olan Cüzdanı İçeri Aktarma
Eğer daha önce kurulum yaptıysanız ya da cüzdanınız var ise aşağı kodu girdikten sonra kurtarma kelmelerinizi girerek cüzdanınızı içe aktarınız.
```
gitopiad keys add "$WALLET" --recover
```

### Validator Oluşturma

```shell
gitopiad tx staking create-validator \
--amount=1000000utlore \
--pubkey=$(gitopiad tendermint show-validator) \
--moniker=validator-ismi \
--chain-id=paloma-testnet-5 \
--commission-rate="0.05" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--fees=5000utlore \
--gas=10000000 \
--from=cuzdan-ismi \
-y
```


## YARARLI KOMUTLAR

## Logları Kontrol Etme 
```
journalctl -fu gitopiad -o cat
```

### Sistemi Başlatma

```
systemctl start gitopiad
```

### Sistemi Durdurma
```
systemctl stop gitopiad
```

### Sistemi Yeniden Başlatma
```
systemctl restart gitopiad
```

### Node Senkronizasyon Durumu
```
gitopiad status 2>&1 | jq .SyncInfo
```
```
curl -s localhost:26657/status | jq .result.sync_info
```

### Validator Bilgileri
```
gitopiad status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri

```
gitopiad status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme

```
gitopiad tendermint show-node-id
```


### Node IP Adresini Öğrenme

```
curl icanhazip.com
```

### Cüzdanların Listesine Bakma

```
gitopiad keys list
```

### Cüzdan Adresini Görme

```
gitopiad keys show $WALLET --bech val -a
```

### Cüzdanı İçeri Aktarma

```
gitopiad keys add $WALLET --recover
```

### Cüzdanı Silme

```
gitopiad keys delete $WALLET
```

### Cüzdan Bakiyesine Bakma

```
gitopiad query bank balances $WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
gitopiad tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 100000000utlore
```

### Proposal Oylamasına Katılma
```
gitopiad tx gov vote 1 yes --from $WALLET --chain-id=$CHAIN_ID
```


### Validatore Stake Etme / Delegate Etme

```
gitopiad tx staking delegate $VALOPER_ADDRESS 100000000utlore --from=$WALLET --chain-id=$CHAIN_ID --gas=auto --fees 5000utlore
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
gitopiad tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 100000000utlore --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

### Ödülleri Çekme

```
gitopiad tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme

```
gitopiad tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CHAIN_ID
```

### Validator İsmini Değiştirme
NEWNODENAME yazan yere yeni validator/moniker isminizi yazınız. TR karakçer içermemelidir.
$WALLET yazan yere de cüzdan adınızı giriniz.
```
seid tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$CHAIN_ID \
--from=$WALLET
```

### Validator Komisyon Oranını Degiştirme
```
gitopiad tx staking edit-validator --commission-rate "0.02" --moniker=$NODENAME --chain-id=$CHAIN_ID --from=$WALLET
```

### Validator Bilgilerinizi Düzenleme
Bu bilgileri değiştirmeden önce https://keybase.io/ adresine kayıt olarak aşağıdaki kodda görüldüğü gibi 16 haneli (XXXX0000XXXX0000) kodunuzu almalısınız. Ayrıca profil resmi vs. ayarları da yapabilirsiniz. 
$NODENAME: Node adınız (moniker)
$WALLET: Cüzdan adınız.
```
gitopiad tx staking edit-validator \
--moniker=$NODENAME \
--identity=XXXX0000XXXX0000 \
--website="VARSA WEBSITENIZI YAZABILIRSINIZ" \
--details="BU BOLUMA KENDINIZI TANITAN BIR CUMLE YAZABILIRSINIZ" \
--chain-id=$CHAIN_ID \
--from=$WALLET
```

### Validatoru Jail Durumundan Kurtarma 

```
gitopiad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CHAIN_ID \
  --gas=auto
  --gas-adjustment=1.4
```

### Node'u Tamamen Silme 

```
systemctl stop gitopiad && \
systemctl disable gitopiad && \
rm /etc/systemd/system/gitopiad.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .gitopia gitopia && \
rm -rf $(which gitopiad)
```

