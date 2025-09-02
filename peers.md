# 🍀 Fresh Peers

```bash
URL="https://atomone-mainnet-rpc.mictonode.com/net_info"
```

```bash
response=$(curl -s $URL)
```

```bash
PEERS=$(echo $response | jq -r '.result.peers[] | select(.remote_ip | test("^[0-9]{1,3}(\\.[0-9]{1,3}){3}$")) | "\(.node_info.id)@\(.remote_ip):" + (.node_info.listen_addr | capture(":(?<port>[0-9]+)$").port)' | paste -sd "," -)
```

```bash
echo "PEERS=\"$PEERS\""
```

```bash
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.atomone/config/config.toml
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart exrpd
sudo journalctl -u exrpd -f -o cat
```

### ➡️ Number of peers to which the node connects

```bash
curl -Ss localhost:${ATOMONE_PORT}657/net_info | jq .result.n_peers
```

### ➡️ You can see the details of your peers with this code

```bash
curl -sS http://localhost:${ATOMONE_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```