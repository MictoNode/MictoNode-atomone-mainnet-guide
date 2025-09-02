# 🔌 Installatio

## 1️⃣ Installation packages and dependencies

```bash
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
sudo apt-get update && apt-get install -y libssl-dev
```

### ➡️ Go Installation

```bash
cd $HOME
VER="1.22.10"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

## 2️⃣ Install node

> **Reminder**  
> You can change the port.

```bash
echo "export ATOMONE_PORT="36"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```bash
cd $HOME
rm -rf atomone
git clone https://github.com/atomone-hub/atomone
cd atomone
git checkout v2.1.0
make build
```

```bash
mkdir -p $HOME/.atomone/cosmovisor/genesis/bin
mv $HOME/atomone/build/atomoned $HOME/.atomone/cosmovisor/genesis/bin/
```

```bash
sudo ln -s $HOME/.atomone/cosmovisor/genesis $HOME/.atomone/cosmovisor/current -f
sudo ln -s $HOME/.atomone/cosmovisor/current/bin/atomoned /usr/local/bin/atomoned -f
```

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
cd
```

### ➡️ Create a service


```bash
sudo tee /etc/systemd/system/atomoned.service > /dev/null << EOF
[Unit]
Description=atomone node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home $HOME/.atomone
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.atomone"
Environment="DAEMON_NAME=atomoned"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.atomone/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

### ➡️ Let's activate it

```bash
sudo systemctl daemon-reload
sudo systemctl enable atomoned
```

### ➡️ Initialize the node

```bash
atomoned init "NODE-NAME" --chain-id atomone-1
```

```bash
sed -i -e '/^chain-id = /c\chain-id = "atomone-1"' $HOME/.atomone/config/client.toml
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${ATOMONE_PORT}657\"|" $HOME/.atomone/config/client.toml
```

### ➡️ Genesis addrbook

> **Info**  
> Updated every 6 hours.

```bash
curl https://files.mictonode.com/atomone/genesis/genesis.json -o ~/.atomone/config/genesis.json
curl https://files.mictonode.com/atomone/addrbook/addrbook.json -o ~/.atomone/config/addrbook.json
```

### ➡️ Port

```bash
sed -i.bak -e "s%:1317%:${ATOMONE_PORT}317%g;
s%:8080%:${ATOMONE_PORT}080%g;
s%:9090%:${ATOMONE_PORT}090%g;
s%:9091%:${ATOMONE_PORT}091%g;
s%:8545%:${ATOMONE_PORT}545%g;
s%:8546%:${ATOMONE_PORT}546%g;
s%:6065%:${ATOMONE_PORT}065%g" $HOME/.atomone/config/app.toml
```

```bash
sed -i.bak -e "s%:26658%:${ATOMONE_PORT}658%g;
s%:26657%:${ATOMONE_PORT}657%g;
s%:6060%:${ATOMONE_PORT}060%g;
s%:26656%:${ATOMONE_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ATOMONE_PORT}656\"%;
s%:26660%:${ATOMONE_PORT}660%g" $HOME/.atomone/config/config.toml
```

### ➡️ Peers and Seeds

```bash
SEEDS=""
PEERS="ed0e36c57122184ab05b6c635b2f2adf592bfa0c@atomone-mainnet-peer.itrocket.net:61657,61b7861a468dfa84532526afd98bea81bf41a874@121.78.247.244:16656,d3adcf9eee8665ee2d3108f721b3613cdd18c3a3@23.227.223.49:26656,3bfb3f122affd7d1b03757b5ea7c44bb1775dd5c@37.27.63.150:23956,00ada7229530d3dd97f0a75f9b8541b809d9dfbd@91.210.101.99:26656,9d79765b925bbaa6a14c12b2efbe762991c141ca@37.120.245.68:26656,eec666a28728bad46a31fd352cc61bc5998efc1d@116.202.156.139:26706,1a956d1315af69d6e08269c15dfa7550cc317089@146.70.243.185:26656,30c45fe5aa7a47e27e568b43ed73f826e08efad4@213.244.249.25:26656,35e145a522e2d81862fdd05fa68296ed8e197d3a@37.27.71.199:23656,13745b16a6e037d9282c7d77980f05f20c4cbd41@152.53.18.245:12656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.atomone/config/config.toml
```

### ➡️ Pruning

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.atomone/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.atomone/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.atomone/config/app.toml
```

### ➡️ Gas Settings

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.025uatone,0.225uphoton"|g' $HOME/.atomone/config/app.toml
```

### ➡️ Prometheus & Indexer

```bash
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.atomone/config/config.toml
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.atomone/config/config.toml
```

### ➡️ Starter Snap

Check snapshot height
```bash
echo "Atomone Snapshot Height: $(curl -s https://files.mictonode.com/atomone/snapshot/block-height.txt)"
```

```bash
exrpd tendermint unsafe-reset-all --home $HOME/.atomone --keep-addr-book

SNAPSHOT_URL="https://files.mictonode.com/atomone/snapshot/"
LATEST_SNAPSHOT=$(curl -s $SNAPSHOT_URL | grep -oP 'atomone_\d+\.tar\.lz4' | sort -t_ -k2 -n | tail -n 1)

if [ -n "$LATEST_SNAPSHOT" ]; then
  FULL_URL="${SNAPSHOT_URL}${LATEST_SNAPSHOT}"
  if curl -s --head "$FULL_URL" | head -n 1 | grep "200" > /dev/null; then
    curl "$FULL_URL" | lz4 -dc - | tar -xf - -C $HOME/.atomone
  else
    echo "Snapshot URL not accessible"
  fi
else
  echo "No snapshot found"
fi

### ➡️ Let's get started

```bash
sudo systemctl start atomoned && sudo journalctl -u atomoned -f -o cat
```

### ➡️ Log Command

```bash
journalctl -u atomoned -f -o cat
```

### ➡️ Create wallet

> **Warning**  
> Don't forget to backup the wallet words!;

```bash
atomoned keys add wallet-name
```

### ➡️ Import wallet

```bash
atomoned keys add wallet-name --recover
```

### ➡️ Create Validator

> **Info**  
> You can't create a validator without Sync. You must have to catch the latest block.

```bash
atomoned tx staking create-validator \
--pubkey $(atomoned tendermint show-validator) \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--amount 1000000uatone \
--moniker "MictoNode" \
--identity "" \
--details "" \
--from wallet-name \
--chain-id atomone-1 \
--gas auto --gas-adjustment 1.5 --fees 60000uphoton \
-y 
```

### ➡️ Delegate to Yourself

```bash
atomoned tx staking delegate $(atomoned keys show wallet-name --bech val -a) amount000000uatone \
--chain-id atomone-1 \
--from "wallet-name" \
--gas auto --gas-adjustment 1.5 --fees 60000uphoton
-y
```

### ➡️ Edit Validator

```bash
atomoned tx staking edit-validator \
--chain-id atomone-1 \
--commission-rate 0.05 \
--new-moniker "validator-name" \
--identity "" \
--details "" \
--website "" \
--security-contact "" \
--from "wallet-name" \
--gas auto --gas-adjustment 1.5 --fees 60000uphoton \
-y
```

### ➡️ Complete deletion

```bash
cd $HOME
sudo systemctl stop atomoned
sudo systemctl disable atomoned
sudo rm -rf /etc/systemd/system/atomoned.service
sudo systemctl daemon-reload
sudo rm -f /usr/local/bin/atomoned
sudo rm -f $(which atomoned)
sudo rm -rf $HOME/.atomone $HOME/atomone
sed -i "/ATOMONE_PORT_/d" $HOME/.bash_profile
```

### ➡️ Block check

```bash
local_height=$(curl -s localhost:${ATOMONE_PORT}657/status | jq -r .result.sync_info.latest_block_height); network_height=$(curl -s https://atomone-mainnet-rpc.mictonode.com/status | jq -r .result.sync_info.latest_block_height); blocks_left=$((network_height - local_height)); echo "Your node height: $local_height"; echo "Network height: $network_height"; echo "Blocks left: $blocks_left"
```

* Your node height - the current block of your node
* Network height - the last block of the network
* Blocks left - how many blocks your node has left to sync.