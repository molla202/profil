📝 Snapshot...
```
sudo apt install curl tmux jq lz4 unzip -y
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.story/story/config/config.toml
```
### 1️⃣ stop node and backup priv_validator_state.json
```
sudo systemctl stop story story-geth
cp $HOME/.story/story/data/priv_validator_state.json $HOME/.story/story/priv_validator_state.json.backup
```
### 2️⃣ remove old data and unpack Story snapshot
```
rm -rf $HOME/.story/story/data
curl http://37.120.189.81/story_dev/story_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/story
```
### 3️⃣ restore priv_validator_state.json
```
mv $HOME/.story/story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json
```
### 4️⃣ delete geth data and unpack Geth snapshot
```
rm -rf $HOME/.story/geth/story/geth/chaindata
curl http://37.120.189.81/story_dev/storygeth_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/geth/odyssey/geth
```
### 5️⃣ restart node and check logs
```
sudo systemctl restart story story-geth
sudo journalctl -u story-geth -u story -f
```
