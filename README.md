




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
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### 📝Dosyaları çekelim
```
echo "export STORY_CHAIN_ID="story"" >> $HOME/.bash_profile
echo "export STORY_PORT="53"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
cd $HOME
wget -O geth https://github.com/piplabs/story-geth/releases/download/v1.0.1/geth-linux-amd64
chmod +x $HOME/geth
mv $HOME/geth $HOME/go/bin/
[ ! -d "$HOME/.story/story" ] && mkdir -p "$HOME/.story/story"
[ ! -d "$HOME/.story/geth" ] && mkdir -p "$HOME/.story/geth"
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
Environment="DAEMON_NAME=story"
Environment="DAEMON_HOME=$HOME/.story/story"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.story/story/data"
ExecStart=$(which cosmovisor) run run
--api-enable \
--api-address=0.0.0.0:${STORY_PORT}317
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
storyd init $MONIKER --chain-id $STORY_CHAIN_ID
storyd config set client chain-id $STORY_CHAIN_ID
storyd config set client node tcp://localhost:${STORY_PORT}657
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
SEEDS="327fb4151de9f78f29ff10714085e347a4e3c836@rpc.story.nodestake.org:666"
PEERS="4761ef729f12b80b3652edd26bd45734b5ff4515@51.15.15.160:26656,30ce6b2ee08c7313a4ef14dbaef0cc6d6937bded@149.50.101.37:26656,22684dfc5f64dc355a1c68d0e4f7472d208caef9@95.216.243.177:26656,de39ffa62ec29003a892218e50e79935d89f1652@34.139.96.9:26656,1b69b89a871cb232300c8a980bfa1584ec1d8a3e@104.196.19.53:26656,1851180d526f7a4cfc5e391263869ba9d24bb8e7@35.211.255.251:26656,a352a98d79cd4b4d9dac83cf8fe1a69d95c81af7@35.211.57.155:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml
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
