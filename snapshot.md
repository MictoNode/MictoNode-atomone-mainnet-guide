# ♻️ Snapshot

> **Info**  
> Updated every 12 hours.  
> Pruning: custom | 100/0/19 | indexer: null

Check snapshot height
```bash
echo "Atomone Snapshot Height: $(curl -s https://files.mictonode.com/atomone/snapshot/block-height.txt)"
```

```bash
sudo systemctl stop atomoned
cp $HOME/.atomone/data/priv_validator_state.json $HOME/.atomone/priv_validator_state.json.backup
rm -rf $HOME/.atomone/data

SNAPSHOT_URL="https://files.mictonode.com/atomone/snapshot/"
LATEST_SNAPSHOT=$(curl -s $SNAPSHOT_URL | grep -oP 'atomone_\d+\.tar\.lz4' | sort -t_ -k2 -n | tail -n 1)

if [ -n "$LATEST_SNAPSHOT" ]; then
  FULL_URL="${SNAPSHOT_URL}${LATEST_SNAPSHOT}"
  if curl -s --head "$FULL_URL" | head -n 1 | grep "200" > /dev/null; then
    curl "$FULL_URL" | lz4 -dc - | tar -xf - -C $HOME/.atomone
    
    mv $HOME/.atomone/priv_validator_state.json.backup $HOME/.atomone/data/priv_validator_state.json
    
    sudo systemctl restart atomoned && sudo journalctl -fu atomoned -o cat
  else
    echo "Snapshot URL is not accessible"
  fi
else
  echo "No snapshot found"
fi
```
