




<h1 align="center"> Story Mainnet


![image](https://github.com/user-attachments/assets/92e39ea8-fc46-48ef-8f61-a27048d3e5f6)



</h1>


 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>


### Explorer And Public Rpc Api



## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	8|
| RAM	| 16+ GB |
| Storage	| 500 GB Nvme-SSD |


### 🚧Gerekli kurulumlar
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip gcc clang cmake build-essential -y
sudo apt install -y \
  curl \
  git \
  make \
  jq \
  build-essential \
  gcc \
  unzip \
  wget \
  lz4 \
  aria2 \
  gh
```

### 🚧 Go kurulumu
```
cd $HOME
GO_VERSION="1.22.0"
wget "https://golang.org/dl/go${GO_VERSION}.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"
rm "go${GO_VERSION}.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

### 📝Dosyaları çekelim
```
echo "export STORY_CHAIN_ID="story"" >> $HOME/.bash_profile
echo "export STORY_PORT="53"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
cd $HOME
git clone https://github.com/piplabs/story-geth
cd story-geth
make geth
cp build/bin/geth $HOME/go/bin/
source $HOME/.bash_profile
[ ! -d "$HOME/.story/story" ] && mkdir -p "$HOME/.story/story"
[ ! -d "$HOME/.story/geth" ] && mkdir -p "$HOME/.story/geth"
```
```
story-geth version
```
```
mkdir -p $HOME/.story/story/cosmovisor/genesis/bin
wget -O $HOME/.story/story/cosmovisor/genesis/bin/storyd https://github.com/piplabs/story/releases/download/v1.0.0/story-linux-amd64
chmod +x $HOME/.story/story/cosmovisor/genesis/bin/storyd
```
### 📝System link
```
sudo ln -s $HOME/.story/story/cosmovisor/genesis $HOME/.story/story/cosmovisor/current -f
sudo ln -s  $HOME/.story/story/cosmovisor/current/bin/storyd /usr/local/bin/storyd -f
```
### 📝Cosmovisor indirelim
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```
### 📝Servis oluşturalım Story
```
sudo tee /etc/systemd/system/storyd.service > /dev/null << EOF
[Unit]
Description=story node service
After=network-online.target

[Service]
User=$USER
Environment="DAEMON_NAME=storyd"
Environment="DAEMON_HOME=$HOME/.story/story"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.story/story/data"
ExecStart=$(which cosmovisor) run run
--api-enable \
--api-address=0.0.0.0:${STORY_PORT}317 \
--network story
Restart=always
RestartSec=5s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### 📝Servis oluşturalım Geth
```
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=${user}
ExecStart=$HOME/go/bin/geth --story --syncmode full --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 0.0.0.0 --http.port ${STORY_PORT}545 --authrpc.port ${STORY_PORT}551 --ws --ws.api eth,web3,net,txpool --ws.addr 0.0.0.0 --ws.port ${STORY_PORT}546
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable storyd story-geth
```
### 📝İnit
NOT: node adınızı yazınız.
```
storyd init $MONIKER --network $STORY_CHAIN_ID
```
### 📝Genesis addrbook 
```
wget -O $HOME/.story/story/config/addrbook.json https://raw.githubusercontent.com/molla202/profil/refs/heads/main/addrbook.json
```

### 📝Gas pruning ayarı
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.story/story/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.story/story/config/config.toml
```

### 📝Port Ayarları

```
sed -i.bak -e "s%:1317%:${STORY_PORT}317%g;
s%:8551%:${STORY_PORT}551%g" $HOME/.story/story/config/story.toml

sed -i.bak -e "s%:26658%:${STORY_PORT}658%g;
s%:26657%:${STORY_PORT}657%g;
s%:26656%:${STORY_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${STORY_PORT}656\"%;
s%:26660%:${STORY_PORT}660%g" $HOME/.story/story/config/config.toml
```
### 📝Seed
```
SEEDS="c1d973eea1b2c637777ab32783b3d37f2b52ba36@b1.storyrpc.io:26656,78db197dbbffb97a5c851b87b1df4cc51e99d4f9@b2.storyrpc.io:26656"
PEERS="55816da1005f3aa89af17bf545a8296058e33e44@164.132.247.253:56356,b1eb613c9026d8643cca4630e4935559bf303d7d@35.211.121.91:26656,c5f37a1293c2baf12e36a0d0f34d1371b1bb576a@35.207.10.148:26656,55f2ea5e1fc7a17000ce7d5adf8ddf7f4c61e4d4@35.207.42.225:26656,155bcba7d521ced31042bd99100841c6cf057f36@35.207.25.245:26656,c45503752041d747c55cad8e9b1ef9896fb1dcf8@141.94.155.97:26656,22684dfc5f64dc355a1c68d0e4f7472d208caef9@95.216.243.177:26656,68c5b1eae074c5b556bf9d32668a9b152ce12b09@35.211.203.203:26656,4761ef729f12b80b3652edd26bd45734b5ff4515@51.15.15.160:26656,1b69b89a871cb232300c8a980bfa1584ec1d8a3e@104.196.19.53:26656,39a4b17027b5eab2968a4e1b974bd0bee6649443@35.237.82.235:26656,de39ffa62ec29003a892218e50e79935d89f1652@34.139.96.9:26656,cd132ca06b578cf4fe2e84b5ba2b49d6eb59944a@35.231.252.72:26656,c0632a90716ec344231f242aba6632a709e25f3b@91.72.69.62:26686,18cdc7c13b2e68a9d97828c1121fc32189ea1767@152.53.125.195:26656,b72e4acda44c0dd281f3a2c3b6c0d31473669271@3.35.213.247:26656,8ede5870a48d1f8edca2568f7db454ba5334e54f@129.213.202.126:26656,b077a58e83f1614b49a66acab786632a97d4ba06@132.145.196.86:26656,8d2fd865d0d1b7537179aa725d4387ef53869b40@47.239.112.90:26656,a13ad94d35632c00f040db03143a179d65870c4a@129.213.125.155:26656,750f7f91e43489da2843a2f207bb3b3d1a9a8ec7@13.52.39.192:26656,97f67874e67693cc20944457e4ef86dc5483c841@152.70.195.0:26656,3ad3cfd89e2a39ce52c23dfb28cf2a732b88d781@150.136.221.45:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+)false$|\1true|" /root/.story/story/config/story.toml
```
### 📝Snap
```
curl -o - -L http://37.120.189.81/story_dev/story_snap.tar.lz4  | lz4 -c -d - | tar -x -C $HOME/.story/story
mkdir -p $HOME/.story/geth/story/geth/
curl -o - -L http://37.120.189.81/story_dev/storygeth_snap.tar.lz4  | lz4 -c -d - | tar -x -C $HOME/.story/geth/story/geth/
```
### 📝Başlatalım   
```
sudo systemctl daemon-reload
sudo systemctl restart story-geth && sleep 5 && sudo systemctl restart storyd
```
### 📝Log
```
sudo journalctl -u storyd.service -f --no-hostname -o cat
sudo journalctl -u story-geth.service -f --no-hostname -o cat
```
### 🔍Cüzdan oluşturma
NOT: cüzdan adınızı yazınız
```
storyd keys add cuzdan-adini-yaz
```
- Eski cüzdan import ederkene bele
```
storyd keys add wallet --recover
```

### 🌟 Validator oluşturma
NOT: çıkan çıktıları kaydedin. dosyalarınızıda yedekleyin.
```
cd $HOME
```
```
story validator export
```
```
story validator export --export-evm-key
```
NOT: alttaki kodla validatorumuzu oluşturalım. ve yedeklerinizi alın.
```
story validator create --stake 1024000000000000000000 --private-key $(cat $HOME/.story/story/config/private_key.txt | grep "PRIVATE_KEY" | awk -F'=' '{print $2}') --moniker "validator-adını-yaz"
```
### Stake
```
story validator stake \
   --validator-pubkey "VALIDATOR_PUB_KEY_IN_HEX" \
   --stake 1024000000000000000000 \
   --private-key $(cat $HOME/.story/story/config/private_key.txt | grep "PRIVATE_KEY" | awk -F'=' '{print $2}')
```
### ⚠️ Delete
```
sudo systemctl stop story story-geth
sudo systemctl disable story story-geth
rm -rf $HOME/.story
sudo rm /etc/systemd/system/story.service /etc/systemd/system/story-geth.service
sudo systemctl daemon-reload
```
