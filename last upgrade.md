# v3.0.0 Update

## BEFORE THE UPGRADE ([BLOCK 5902000](https://explorer.mictonode.com/Atomone-Mainnet/block/5902000))

### Preparing for the Upgrade

#### 1. Create Upgrade Directories and Download the Binary

```bash
cd $HOME
mkdir -p $HOME/.atomone/cosmovisor/upgrades/v3/bin
cd $HOME
rm -rf atomone
git clone https://github.com/atomone-hub/atomone
cd atomone
git checkout v3.0.0
make build
mv $HOME/atomone/build/atomoned $HOME/.atomone/cosmovisor/upgrades/v3/bin/
cd $HOME
```

#### 2. Check Version

```bash
$HOME/.atomone/cosmovisor/upgrades/v3/bin/atomoned version
```

> **Note:** If the version shows as `v3.0.0`, the preparation is complete.

---

## AFTER THE UPGRADE ([BLOCK 5902000](https://explorer.mictonode.com/Atomone-Mainnet/block/5902000))

#### 1. Stop the Service and Download the Binary

```bash
cd $HOME
mkdir -p $HOME/.atomone/cosmovisor/upgrades/v3/bin
cd $HOME
rm -rf atomone
git clone https://github.com/atomone-hub/atomone
cd atomone
git checkout v3.0.0
make build
mv $HOME/atomone/build/atomoned $HOME/.atomone/cosmovisor/upgrades/v3/bin/
cd $HOME
```

#### 2. Activate the New Version

```bash
sudo ln -sfn $HOME/.atomone/cosmovisor/upgrades/v3 $HOME/.atomone/cosmovisor/current
sudo ln -sfn $HOME/.atomone/cosmovisor/current/bin/atomoned /usr/local/bin/atomoned
```

#### 3. Restart and Monitor the Service

```bash
sudo systemctl restart atomoned
sudo journalctl -fu atomoned -o cat
```