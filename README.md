<h1 align="center"> MandeNetwork-Testnet</h1>

![image](https://user-images.githubusercontent.com/101191449/198585174-42c7aa0a-77bc-426d-9112-de948cdeb979.png)

Bu rehberde https://github.com/CryptoniteNodes/Testnet-Mande referans olarak kullanılmıştır.

## Sistem gereksinimleri
```
2-4 CPU
4 GB RAM 
100 GB SSD
```
# Linkler
> ## [Discord](https://discord.com/invite/9Ugch3fRC2)<br>
> ## [Mande website](https://www.mande.network/)

# Node Kurulumu Rehberi

## root yetkisi kazanıyoruz.
```
sudo su
```

## root dizinine gidiyoruz.
```
cd /root
```

## Sistem güncellemesi yapıyoruz.
```
sudo apt update && sudo apt upgrade -y
```


## Kütüphane kurulumu yapıyoruz.
```
sudo apt install curl tar wget tmux htop net-tools clang pkg-config libssl-dev jq build-essential git make ncdu -y
```


## Go kurulumu yapıyoruz.
```
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```


## Mande Network'un github dosyasını Mande Network repo'sundan sunucumuza çekiyoruz.
```
cd $HOME
curl -OL https://github.com/mande-labs/testnet-1/raw/main/mande-chaind
mv mande-chaind /usr/local/bin
chmod 744 /usr/local/bin/mande-chaind
```

## initialize (başlatma) işlemini yapıyoruz.NodeName kısmına kendi validator ismimizi giriyoruz.
```
mande-chaind init Nodename --chain-id mande-testnet-1
```


## Cüzdan oluşturuyoruz.walletName kısmına kendi cüzdan ismimizi yazıyoruz. Seed bilgisini kaydetmeyi unutmayın
```
mande-chaind keys add walletName
```


## Genesis Dosyaları İndiriyoruz
```
wget -O $HOME/.mande-chain/config/genesis.json "https://raw.githubusercontent.com/mande-labs/testnet-1/main/genesis.json"
```


## Minimum Gas ücreti, Seed ve Peer'ları düzenliyoruz.
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0mand\"/;" ~/.mande-chain/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.mande-chain/config/config.toml
peers="f9dec9a209fdcb22339d87eaadffde97d3ecf648@45.67.216.40:26656,c2ec4f71950d1d4e6233ed450b09f08d15ffbe98@195.201.165.123:10086,4fbbf09c5561c4a9692e368a672b99180b3f70ee@185.182.184.200:46656,156eb9c408b5274c14e7139fa14b3210de359848@5.161.113.160:26656,074a8eaf817da9df97c5becf367baaf2f3e1917f@135.125.163.63:26666,19a7467dc9aa99b3cdc8ee82492a57c4ffa46fc3@5.161.98.239:26656,2365cf1278df6bdf26b314d4f9c4e4108734b51d@144.126.156.253:26656,a3e3e20528604b26b792055be84e3fd4de70533b@38.242.199.93:24656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mande-chain/config/config.toml
seeds="cd3e4f5b7f5680bbd86a96b38bc122aa46668399@34.171.132.212:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.mande-chain/config/config.toml
```



## Pruning açıyoruz. (Disk kullanımını düşürür - cpu ve ram kullanımını arttırır) 
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.mande-chain/config/app.toml
```


## İndexer kapatıyoruz (Disk kullanımını düşürür) 
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mande-chain/config/config.toml
```


## addrbook Dosyasını İndiriyoruz. 
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mande-chain/config/config.toml
```


## Servis dosyası oluşturuyoruz başlatıyoruz
```
sudo tee /etc/systemd/system/mande-chaind.service > /dev/null <<EOF
[Unit]
Description=mande-chaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mande-chaind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Node'umuzu başlatıyoruz. 
```
sudo systemctl daemon-reload
sudo systemctl enable mande-chaind
sudo systemctl restart mande-chaind && sudo journalctl -u mande-chaind -f -o cat
```

## StateSync(Hızlı sync olmak istiyorsanız yapabilirsiniz. Zorunlu bir işlem değil)
```
SNAP_RPC=http://38.242.199.93:24657
peers="a3e3e20528604b26b792055be84e3fd4de70533b@38.242.199.93:24656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.mande-chain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.mande-chain/config/config.toml

mande-chaind tendermint unsafe-reset-all --home /root/.mande-chain --keep-addr-book
systemctl restart mande-chaind && journalctl -u mande-chaind -f -o cat
```


## Faucetten token talep ediyoruz. Walletadress yazan yere kendi cüzdan adresinizi girin. 
```
curl -d '{"address":"Walletadress"}' -H 'Content-Type: application/json' http://35.224.207.121:8080/request
```

Eğer Faucet Çalışmazsa önce terminalin size verdiği seed bilgisi ile Keplr Wallet'tan cüzdanınızı import ettikten sonra http://35.224.207.121/ adresine giderek import ettiğiniz cüzdanı platforma bağlayın ve token talebinde bulunun


## Sync olduktan sonra son adım olarak validator oluşturuyoruz.
```
mande-chaind tx staking create-validator \
--chain-id mande-testnet-1 \
--amount 0cred \
--pubkey "$(mande-chaind tendermint show-validator)" \
--from WalletName \
--moniker="NodeName" \
--fees 1000mand
```


## Sync durumuna bakmak için. (False çıktısı sync olduğunuz anlamına gelir.)
```
mande-chaind status 2>&1 | jq .SyncInfo
```

# Explorer
> ## [STAVR](https://explorer.stavr.tech/mande-chain/)<br>
> ## [JAMBULMERAH](https://explorer.jambulmerah.dev/mande-testnet/)

