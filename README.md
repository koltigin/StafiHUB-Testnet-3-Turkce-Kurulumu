# StaFiHub Public Testnet v3

![image](https://user-images.githubusercontent.com/102043225/178576674-8e7e8abf-650c-4eaf-8f50-e2fcc283d829.png)


## Sistem Gereksinimleri
* **Minimum**
  * 4GB RAM
  * 200GB SSD
  * 2 vCPU
* **Önerilen**
  * 8GB RAM
  * 300GB SSD
  * 4 vCPU

## Sistemi Güncelleme
```shell
cd $HOME
sudo apt update
```

## Gerekli Kütüphanelerin Kurulması
```shell
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"
```
## Go Kurulumu
```shell
cd $HOME
wget -O go1.18.2.linux-amd64.tar.gz https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```

## Stafi HUB Dosyaların İndirilmesi
```shell
git clone --branch public-testnet-v3 https://github.com/stafihub/stafihub
```

## Stafi HUB Yüklenmesi
```shell
cd $HOME/stafihub && make install
```

## Genesis Dosyasının İndirilmesi 
* Bu bölüme `NODE_ADINIZ` kendi node adınızı yazınız.:
```shell
stafihubd init NODE_ADINIZ --chain-id stafihub-public-testnet-3
wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/testnets/stafihub-public-testnet-3/genesis.json"
stafihubd tendermint unsafe-reset-all --home ~/.stafihub
```
## Node'un Yapılandırılması
```shell
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
peers="4e2441c0a4663141bb6b2d0ea4bc3284171994b6@46.38.241.169:26656,79ffbd983ab6d47c270444f517edd37049ae4937@23.88.114.52:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```

## Indexer'i İnaktif Etme
* Pruning disk kullanımını düşürür.
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stafihub/config/config.toml
```

## Pruning Yapılması 
* Pruning disk kullanımını düşürür, CPU ve RAM kullanımını arttırır. Eğer SSD sıkıntısınız yoksa yapmamanız tavsiye edilir.
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stafihub/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stafihub/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stafihub/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stafihub/config/app.toml
```

## Servis Dosyası Oluşturma ve Node'u Başlatma
Aşağıdaki kodu tek seferde giriniz.

```shell
echo "[Unit]
Description=StaFiHub Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which stafihubd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/stafihubd.service
sudo mv $HOME/stafihubd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable stafihubd
sudo systemctl restart stafihubd
```

## Logları Kontrol Etme
```shell
journalctl -u stafihubd -f
```

## Cüzdan Oluşturma
### Yeni Cüzdan Oluşturma
`CUZDAN_ADINIZ` bölümüne cüzdanınızın ismini yazın.
```shell
stafihubd keys add CUZDAN_ADINIZ --keyring-backend file
```
###Var Olan Cüzdanı İçeri Aktarma
stafihubd keys add CUZDAN_ADINIZ --recover

## Faucet / Musluk
Test token almak için Discord'da [#faucet](https://discord.gg/a6CMqMh47e) kanalından aşağıdaki şekilde mesaj atıyoruz.
```shell
!faucet send CUZDAN_ADRESINIZ
```

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki adıma yani validator oluşturma adımına geçmiyoruz.
```shell
stafihubd status 2>&1 | jq .SyncInfo
```

## Validator Oluşturma
* Aşağıdaki komutta `NODE_ADINIZ` ve `CUZDAN_ADINIZ` bölümlerini değiştiriyoruz.
```shell
stafihubd tx staking create-validator -y --amount=1000000ufis --pubkey=$(stafihubd tendermint show-validator) --moniker=NODE_ADINIZ --commission-rate=0.10 --commission-max-rate=0.20 --commission-max-change-rate=0.01 --min-self-delegation=1 --from=CUZDAN_ADINIZ --chain-id=stafihub-public-testnet-3 --gas-prices=0.025ufis --keyring-backend file
```

## Explorer
Node'unuzu [explorer](https://testnet-explorer.stafihub.io)'dan takip edebilirsiniz.
