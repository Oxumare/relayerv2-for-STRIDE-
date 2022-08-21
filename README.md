# relayerv2-for-STRIDE-
Merhabalar,bugün Stride TestNet için RelayerV2 aktarıcıyı yükleyip çalıştıracağız.Unutmamız gereken ilk önemli konu elimize fullnode çalışan(full sync) iki chain olmalıdır.

 **1.Stride Node   2.Gaia Node**
 
 Aşağıdaki kurulum adımları yukarıda yazılan şartlar dahilinde uygulanmalıdır.
 
 İşlemlerimize sırasıyla başlayalım;

 **Indexer Açma**

 Stride node içine giriyoruz ve aşağıdaki indexer komutunu uyguluyoruz;
 
 ```
 sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.stride/config/config.toml
 ```
 
 Gaia node içine giriyoruz ve aşağıdaki indexer komutunu uyguluyoruz;
 
 ```
 sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.gaia/config/config.toml
 ```
 
 **Değişkenleri Ayarlayalım**

 Stride node içine giriyoruz ve aşağıdaki komutunu uygulayarak bize lazım olan IP ve Port bilgilerini alıp bir kenara not edelim;
 
 ```
 nano $HOME/.stride/config/config.toml
 ```
 
 Aşağıdaki komutu uygulayalım;
 
 ```
 STRIDE_RPC_ADDR='' # Not ettiğiniz IP ve Port yazalım
 STRIDE_MNEMONIC='' # Mnemonic kelimelerinizi yazın
 ```
 
 Gaia node içine giriyoruz ve aşağıdaki komutunu uygulayarak bize lazım olan IP ve Port bilgilerini alıp bir kenara not edelim;
 
 ```
 nano $HOME/.gaia/config/config.toml
 ```
 
 Aşağıdaki komutu uygulayalım;
 
 ```
 GAIA_RPC_ADDR='' # Not ettiğiniz IP ve Port yazalım
 GAIA_MNEMONIC='' # Mnemonic kelimelerinizi yazın
 ```
 
 Stride node içine geliyoruz;
 
```
RELAYER_ID='ABRAHAM#7979'         # Kendi Discord Adınızı yazın
STRIDE_RPC_ADDR='127.0.0.1:16657' # Yukarıdaki adımda not ettiğiniz IP ve Port yazın(Stride)
STRIDE_MNEMONIC=''                # Mnemonic kelimelerinizi yazın
GAIA_RPC_ADDR='127.0.0.1:23657'   # Yukarıdaki adımda not ettiğiniz Ip ve Port yazın(Gaia)
GAIA_MNEMONIC=''                  # Mnemonic kelimelerinizi yazın
```

**Sistemi Güncelleyelim**
```
sudo apt update && sudo apt upgrade -y
```

**Go Yükleyelim**
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.3"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

**Go relayerV2 kurup başlatalım**
```
git clone https://github.com/cosmos/relayer.git
cd relayer && git checkout v2.0.0-rc4
make install
rly config init --memo $RELAYER_ID
sudo mkdir $HOME/.relayer/chains
sudo mkdir $HOME/.relayer/paths
```

**Stride .json dosyası oluşturalım**

.json dosyası  oluştururken key kısmına cüzdan adımızı yazalım !

```
sudo tee $HOME/.relayer/chains/stride.json > /dev/null <<EOF
{
  "type": "cosmos",
  "value": {
    "key": "wallet",
    "chain-id": "STRIDE-TESTNET-4",
    "rpc-addr": "http://${STRIDE_RPC_ADDR}",
    "account-prefix": "stride",
    "keyring-backend": "test",
    "gas-adjustment": 1.2,
    "gas-prices": "0.000ustrd",
    "gas": 200000,
    "timeout": "20s",
    "trusting-period": "8h",
    "output-format": "json",
    "sign-mode": "direct"
  }
}
EOF
```

**Gaia .json dosyası oluşturalım**

.json dosyası  oluştururken key kısmına cüzdan adımızı yazalım !

```
sudo tee $HOME/.relayer/chains/gaia.json > /dev/null <<EOF
{
  "type": "cosmos",
  "value": {
    "key": "wallet",
    "chain-id": "GAIA",
    "rpc-addr": "http://${GAIA_RPC_ADDR}",
    "account-prefix": "cosmos",
    "keyring-backend": "test",
    "gas-adjustment": 1.2,
    "gas-prices": "0.000uatom",
    "gas": 200000,
    "timeout": "20s",
    "trusting-period": "8h",
    "output-format": "json",
    "sign-mode": "direct"
  }
}
EOF
```

**Relayer zincirlerini yükleyelim**
```
rly chains add --file=$HOME/.relayer/chains/stride.json stride
rly chains add --file=$HOME/.relayer/chains/gaia.json gaia
```

Zincir kontrolü sağlayalım;

```
rly chains list
```

Eğer işlemlerimiz başarılı ise aşağıdaki çıktıyı almalıyız;

```1: GAIA             -> type(cosmos) key(✘) bal(✘) path(✘)```
```2: STRIDE-TESTNET-4 -> type(cosmos) key(✘) bal(✘) path(✘)```


**Cüzdanlarımzı aktarıcıya yükleyelim**

$wallet kısmına cüzdan adımzı yazalım

```
rly keys restore stride $wallet "your stride wallet mnemonic goes here"
rly keys restore gaia $wallet "your gaia wallet mnemonic goes here"
```

Cüzdan bakiyemizi kontrol edelim;

```
rly q balance stride
rly q balance gaia
```

**Stride-Gaia arasında yollar .json dosyası oluşturalım**

```
sudo tee $HOME/.relayer/paths/stride-gaia.json > /dev/null <<EOF
{
  "src": {
    "chain-id": "STRIDE-TESTNET-4",
    "client-id": "07-tendermint-0",
    "connection-id": "connection-0"
  },
  "dst": {
    "chain-id": "GAIA",
    "client-id": "07-tendermint-0",
    "connection-id": "connection-0"
  },
  "src-channel-filter": {
    "rule": "allowlist",
    "channel-list": ["channel-0", "channel-1", "channel-2", "channel-3", "channel-4"]
  }
}
EOF
```

Yol ekleyelim;

```
rly paths add STRIDE-TESTNET-4 GAIA stride-gaia --file $HOME/.relayer/paths/stride-gaia.json
```

Bu yolu kontrol edelim;

```
rly paths list
```

Eğer işlemlerimiz başarılı ise aşağıdaki çıktıyı almalıyız;

```0: stride-gaia -> chns(✔) clnts(✔) conn(✔) (STRIDE-TESTNET-4<>GAIA)```

**Hizmeti oluşturalım**

```
sudo tee /etc/systemd/system/relayerd.service > /dev/null <<EOF
[Unit]
Description=GO Relayer v2 Service
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which rly) start stride-gaia -p events
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

**!_ Buraya bir not düşelim. Eğer Stride Nodenizde çalışan başka bir aktarıcı var ise (hermes vb) bu hizmeti durdurmalısınız**_!

**Hizmeti başlatalım**

```
sudo systemctl daemon-reload
sudo systemctl enable relayerd
sudo systemctl start relayerd
```

**Logları kontrol edelim**

```
journalctl -u relayerd -f -o cat
```


**Artık aktarıcı (relayerv2) kullanarak işlem gerçekleştirebiliriz.**

**Örnek Stride>Gaia ve Gaia>Stride**

```
rly transact transfer stride gaia 1000ustrd cosmosadresiniziyazın channel-0 --path stride-gaia
```

```
rly transact transfer gaia stride 1000uatom strideadresiniziyazın channel-0 --path stride-gaia
```

İşlemleriniz başarılı şekilde gerçekleşirse aşağıdaki görntüye sahip olacaktır.Stride explorerden kontrol ediniz.

![11](https://user-images.githubusercontent.com/43583832/185770322-2b4b01d8-8435-4d3f-8b1b-f13e8fe785c9.png)

![22](https://user-images.githubusercontent.com/43583832/185770424-44824b76-cb24-4018-95a7-7e02e9f4a35d.png)






