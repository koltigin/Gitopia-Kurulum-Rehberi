# Gitopia Türkçe Kurulum Rehberi
[Kaynak](https://docs.gitopia.com/validator-setup/index.html) / [Explorer](https://explorer.gitopia.com/)
![image](https://user-images.githubusercontent.com/102043225/176181117-e8589025-8649-4ac3-9308-08c48f0335fe.png)

## Bağlantılar
 * [Gitopia Website](https://gitopia.com/)
 * [Gitopia Discord](https://discord.gg/GhkA9SGJhD)
 * [Gitopia Türkçe Destek](https://t.me/GitopiaTR)

## Gereksinimler 
| Bileşenler | Minimum Gereksinimler | **Tavsiye Edilen Gereksinimler** | 
| ------------ | ------------ | ------------ |
| CPU |	4 | Intel Core i7-8700 Hexa-Core |
| RAM	| 8 GB | 32 GB |
| Storage	| 400 GB SSD | 1 TB SSD |

## Sistemi Güncelleme
```shell
sudo apt update && sudo apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
sudo apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen gcc -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.19"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Değişkenleri Yükleme
aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$GITOPIA_NODENAME` validator adınız
* `$GITOPIA_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export GITOPIA_NODENAME=$GITOPIA_NODENAME"  >> $HOME/.bash_profile
echo "export GITOPIA_WALLET=$GITOPIA_WALLET" >> $HOME/.bash_profile
echo "export GITOPIA_PORT=16" >> $HOME/.bash_profile
echo "export GITOPIA_CHAIN_ID=gitopia-janus-testnet-2" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın Mehmet olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export GITOPIA_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export GITOPIA_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export GITOPIA_PORT=18" >> $HOME/.bash_profile
echo "export GITOPIA_CHAIN_ID=Testnet3" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Gitopia'nın Kurulması

```shell
curl https://get.gitopia.com | bash
git clone -b v1.2.0 gitopia://gitopia/gitopia
cd gitopia && make install
```

Versiyon Kontrolü
```shell
gitopiad version --long
```
Çıktı aşağıdaki gibi olmalıdır; 
```shell
build_deps:
- cloud.google.com/go@v0.102.1
- cloud.google.com/go/compute@v1.7.0
- cloud.google.com/go/iam@v0.4.0
...
build_tags: netgo,ledger
commit: 9bf1b1d5211f7afc0ca42fb4b7a411b0437a301f
cosmos_sdk_version: v0.46.3
go: go version go1.18.6 darwin/arm64
name: gitopia
server_name: gitopiad
version: 1.2.0
```

## Uygulamayı Yapılandırma ve Başlatma
```shell
gitopiad config chain-id $GITOPIA_CHAIN_ID
gitopiad init --chain-id $GITOPIA_CHAIN_ID $GITOPIA_NODENAME
```

## Genesis Dosyasının Kopyalanması
```shell
wget https://server.gitopia.com/raw/gitopia/testnets/master/gitopia-janus-testnet-2/genesis.json.gz
gunzip genesis.json.gz
mv genesis.json $HOME/.gitopia/config/genesis.json
```

Genesis dosyasını doğrulama;
```shell
shasum -a 256 $HOME/.gitopia/config/genesis.json
```
Yukarıdaki kodun çıktısı şu şekilde olmalıdır; `18961495f3b65a22928b98cc0b98073e39d3e8b17ee369f087ffe3ab2cd970e9`

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001utlore\"/" $HOME/.gitopia/config/app.toml
```

## Indexer'i Kapatma (Opsiyonel)
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gitopia/config/config.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS="399d4e19186577b04c23296c4f7ecc53e61080cb@seed.gitopia.com:26656"
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.gitopia/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gitopia/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gitopia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gitopia/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gitopia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gitopia/config/app.toml
```

## Portları Ayarlama
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${GITOPIA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${GITOPIA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${GITOPIA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${GITOPIA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${GITOPIA_PORT}660\"%" $HOME/.gitopia/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${GITOPIA_PORT}317\"%; s%^address = \":8080\"%address = \":${GITOPIA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${GITOPIA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${GITOPIA_PORT}091\"%" $HOME/.gitopia/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${GITOPIA_PORT}657\"%" $HOME/.gitopia/config/client.toml
```

## Servis Dosyası Oluşturma
```shell
mv $HOME/go/bin/gitopiad /usr/bin/
tee <<EOF >/dev/null /etc/systemd/system/gitopiad.service
[Unit]
Description=Gitopia
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gitopiad) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Servisi Başlatma
```shell
systemctl daemon-reload
systemctl enable gitopiad
systemctl restart gitopiad
```

## Logları Kontrol Etme
```shell
journalctl -u gitopiad -f -o cat
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$GITOPIA_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
gitopiad keys add $GITOPIA_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
gitopiad keys add $GITOPIA_WALLET --recover
```

## Cüzdan ve Valoper Bilgileri
Burada cüzdan ve valoper bilgilerimizi değişkene ekliyoruz.

```shell
GITOPIA_WALLET_ADDRESS=$(gitopiad keys show $GITOPIA_WALLET -a)
GITOPIA_VALOPER_ADDRESS=$(gitopiad keys show $GITOPIA_WALLET --bech val -a)
```

```shell
echo 'export GITOPIA_WALLET_ADDRESS='${GITOPIA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export GITOPIA_VALOPER_ADDRESS='${GITOPIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

🔴 **Cüzdan oluşturma ya da içeri aktarma sırasında aşağıdaki gibi bir hata alırsanız**

 * `-bash: /root/go/bin/gitopiad: No such file or directory`
Bu kodu giriniz;
```shell
cp /usr/bin/gitopiad /root/go/bin
```

## Faucet
[Gitopia](https://gitopia.com/home) adresine giderek yukarıda oluşturduğumuz cüzdanı kepler ile siteye bağlayarak `Get TLORE` butonuna basarak 10 adet token istiyoruz. 

🔴 **BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.**

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
gitopiad status 2>&1 | jq .SyncInfo
```

🔴 **Eşleşme tamamlandıysa aşağıdaki adıma geçiyoruz.**


## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişiklik yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere [keybase](https://keybase.io/) sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
gitopiad tx staking create-validator \
--amount=7000000utlore \
--pubkey=$(gitopiad tendermint show-validator) \
--moniker=$GITOPIA_NODENAME \
--chain-id=$GITOPIA_CHAIN_ID \
--commission-rate="0.1" \
--commission-max-rate="0.5" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees=250utlore \
--gas=200000 \
--from=$GITOPIA_WALLET \
--details="Always forward with the Anatolian Team 🚀" \
--security-contact="xxxxxxx@gmail.com" \
--website="https://anatolianteam.com" \
--identity="XXXX1111XXXX1111" \
--yes
 ``` 

## Form Doldurma
Validator oluşturduktan sonra [buradaki](https://airtable.com/shrMQFJxcsMD0XV2M) formu da doldurunuz.

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
gitopiad keys show $GITOPIA_WALLET --bech val -a
```

### Cüzdanı İçeri Aktarma

```
gitopiad keys add $GITOPIA_WALLET --recover
```

### Cüzdanı Silme

```
gitopiad keys delete $GITOPIA_WALLET
```

### Cüzdan Bakiyesine Bakma

```
gitopiad query bank balances $GITOPIA_WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
gitopiad tx bank send $GITOPIA_WALLET_ADDRESS GONDERILECEK_CUZDAN_ADRESI 100000000utlore
```

### Proposal Oylamasına Katılma
```
gitopiad tx gov vote 1 yes --from $GITOPIA_WALLET --chain-id=$GITOPIA_CHAIN_ID
```


### Validatore Stake Etme / Delegate Etme

```
gitopiad tx staking delegate $VALOPER_ADDRESS 100000000utlore --from=$GITOPIA_WALLET --chain-id=$GITOPIA_CHAIN_ID --gas=auto --fees 5000utlore
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
gitopiad tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 100000000utlore --from=$GITOPIA_WALLET --chain-id=$GITOPIA_CHAIN_ID --gas=auto
```

### Ödülleri Çekme

```
gitopiad tx distribution withdraw-all-rewards --from=$GITOPIA_WALLET --chain-id=$GITOPIA_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme

```
gitopiad tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$GITOPIA_WALLET --commission --chain-id=$GITOPIA_CHAIN_ID
```

### Validator İsmini Değiştirme
NEWNODENAME yazan yere yeni validator/moniker isminizi yazınız. TR karakçer içermemelidir.

```
seid tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$GITOPIA_CHAIN_ID \
--from=$GITOPIA_WALLET
```

### Validator Komisyon Oranını Degiştirme
```
gitopiad tx staking edit-validator --commission-rate "0.02" --moniker=$GITOPIA_NODENAME --chain-id=$GITOPIA_CHAIN_ID --from=$GITOPIA_WALLET
```

### Validator Bilgilerinizi Düzenleme
Bu bilgileri değiştirmeden önce https://keybase.io/ adresine kayıt olarak aşağıdaki kodda görüldüğü gibi 16 haneli (XXXX0000XXXX0000) kodunuzu almalısınız. Ayrıca profil resmi vs. ayarları da yapabilirsiniz. 
$NODENAME: Yeni node adınız (moniker)
$GITOPIA_WALLET: Cüzdan adınız, değiştirmeniz gerekmez. Değişkenlere ekledik çünkü.
```
gitopiad tx staking edit-validator \
--moniker=$NODENAME \
--identity=XXXX0000XXXX0000 \
--website="VARSA WEBSITENIZI YAZABILIRSINIZ" \
--details="BU BOLUME KENDINIZI TANITAN BIR CUMLE YAZABILIRSINIZ" \
--chain-id=$GITOPIA_CHAIN_ID \
--from=$GITOPIA_WALLET
```

### Validatoru Jail Durumundan Kurtarma 

```
gitopiad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$GITOPIA_CHAIN_ID \
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
rm -rf $(which gitopiad) \
sed -i '/GITOPIA_/d' ~/.bash_profile
```

# Hesaplar:

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
