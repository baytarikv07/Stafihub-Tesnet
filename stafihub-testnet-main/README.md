# <h1 align="center">StaFiHub Public Testnet 3</h1>

<h1 align="center">Selamlar herkese, StaFiHub'un public validator testine katılacağız, daha önce katılmadıysanızda dahil olabilirsiniz. </h1>

![image](https://user-images.githubusercontent.com/101149671/177503012-d32a16d9-943b-4090-a005-dccaa691cb46.png)

## Öncelikle telegram kanalını bırakayım: https://t.me/StaFiHubTurkish

Not: Bu kaynağı sağ üstten forklamayı ve yıldızlamayı unutmayın, profiliniz katıldığınız testnetlerle dolu olsun.

# Sistem gereksinimler: NOT: bence ücretsiz sunucuya kurun aşağıda purring ve index ayarlarını yaparak
```
8GB RAM
200 GB SSD
4 vCPU
```

# Stafihub tesnet - 2 katılanlar öncelikle dosyalarını silsin:
```
sudo systemctl stop stafihubd && \
sudo systemctl disable stafihubd && \
rm /etc/systemd/system/stafihubd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .stafihub stafihub && \
rm -rf $(which stafihubd)
```

# Başlıyoruz:
```
sudo su
```

# root dizinine gidiyoruz:
```
cd /root
```

# Sistem güncellemesi yapıyoruz:
```
sudo apt update && sudo apt upgrade -y
```

# Kütüphane kurulumu yapıyoruz:
```
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"
```

# Go kurulumu yapıyoruz:
```
cd $HOME
wget -O go1.18.2.linux-amd64.tar.gz https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```

# Stafi'nin github dosyasını klonluyoruz
```
git clone --branch public-testnet-v3 https://github.com/stafihub/stafihub
```

# Kurulum:
```
cd $HOME/stafihub && make install
```

# initialize işlemimi, nodename kısmına kendi isminizi girin
```
stafihubd init NodeName --chain-id stafihub-public-testnet-3
```

# Genesis file indiriyoruz:
```
wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/testnets/stafihub-public-testnet-3/genesis.json"
```

# Node çin gerekli configurasyonları yapıyoruz:
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
peers="2e48b16575a278b90b605cb243a48c909de7fd03@194.60.201.136:26656,c8ba1915c78db452e56d50d3ba4d3357aeeda2d0@161.97.82.204:26656,c4509887f5eaea749bcdb5659aefbe857062d47c@142.93.170.65:26656,4e2441c0a4663141bb6b2d0ea4bc3284171994b6@46.38.241.169:26656,79ffbd983ab6d47c270444f517edd37049ae4937@23.88.114.52:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```

# Pruning açıyoruz. (Disk kullanımını düşürür - cpu ve ram kullanımı artar) (Kişisel Tercih)
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stafihub/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stafihub/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stafihub/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stafihub/config/app.toml
```

# İndexer kapatırız (Disk kullanımını düşürür) (Kişisel Tercih)
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.stafihub/config/config.toml
```

# Servis dosyası oluşturup node'umuzu başlatıyoruz
```
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

# Cüzdan oluşturmak için. walletName kısmına kendi cüzdan isminizi yazın:
```
stafihubd keys add walletName
```

# Eskiden katıldıysanız recover:
```
stafihubd keys add walletName --recover
```

# Cüzdanı hallettikten sonra Faucet için: https://discord.gg/tz6USZWX

![image](https://user-images.githubusercontent.com/101149671/177530840-d825e54c-fb2e-4c2f-8349-7d534d9be095.png)

## NOT: Peer listesi en güncel halde, bizzat 2 sunucuda denedik ve maks 10-15dk alıyor, eşleşemezseniz bile biraz zaman tanıyın lütfen.

# Eşleşme komudu, bu komudu girdğinizde en altta false yazması lazım, örnek:
```
stafihubd status 2>&1 | jq .SyncInfo
```

# Şu an true yazıyor: 

![image](https://user-images.githubusercontent.com/101149671/177506402-4938b31e-1a3c-417d-bb35-c8b1066f03ff.png)

# Eşleşince: böyle bir çıktı alırsınız.

![image](https://user-images.githubusercontent.com/101149671/177517503-12f2dfea-9126-4ae9-b74d-92c456bff7ce.png)


# Validator oluşturma: 
```
stafihubd tx staking create-validator \
--moniker="NodeName" \
--amount=48885000ufis \
--gas auto \
--fees=5000ufis \
--pubkey=$(stafihubd tendermint show-validator) \
--chain-id=stafihub-public-testnet-3 \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.10 \
--min-self-delegation=1 \
--from=WalletName \
--yes
```

# Oluştuktan sonra böyle bir çıktı gelir, tx hash'ı kopyalayıp explorerda aratın, succes demesi lazım.

![image](https://user-images.githubusercontent.com/101149671/177530977-9e98bcbd-5928-41d8-9a9a-afde3b0a1a85.png)

![image](https://user-images.githubusercontent.com/101149671/177531070-e170759e-4996-44eb-bd46-1b6e4ff4c868.png)


# Yararlı komutlar için konu başlığı: https://forum.rues.info/index.php?threads/stafihub-testnet-3-katilim-rehberi.2074/

# Explorer linki: https://testnet-explorer.stafihub.io/stafi-hub-testnet
# Telegram kanalı: https://t.me/StaFiHubTurkish
























