# PANDUAN TESNET NEAR - STAKEWARS III

Link Dokumen Official
- [ Website ](https://near.org/stakewars/)
- [ Twitter ](https://twitter.com/nearprotocol)
- [ Discord ](https://discord.com/invite/UY9Xf2k)
- [ Telegram ](https://t.me/cryptonear)
- [ Instruksi Setting](https://github.com/near/stakewars-iii/blob/main/challenges/001.md)
- [ Explorer ](https://explorer.shardnet.near.org/)

Sebelum memulainya, anda dapat membeli vps di [Digital Ocean](https://www.digitalocean.com/) dengan spesifikan 4 cpu dan ram 8GB / 4cpu dan ram 16GB untuk hasil yang lebih optimal dengan harga kurang lebih $56

# Cara Installasi
- Buat wallet terlebih dahulu: https://wallet.shardnet.near.org/

# Update & download
```sh
sudo apt update && sudo apt upgrade -y

sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker protobuf-compiler libssl-dev pkg-config clang llvm cargo clang build-essential make

sudo apt install python3-pip 

USER_BASE_BIN=$(python3 -m site --user-base)/bin export PATH="$USER_BASE_BIN:$PATH"
```

# Install Node JS and NPM
```sh
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash - 
sudo apt install build-essential nodejs 
PATH="$PATH"
```

# Install NEAR Cli

```sh
sudo npm install -g near-cli
```

# Set up the environment.
```sh
export NEAR_ENV=shardnet 
echo ‘export NEAR_ENV=shardnet’ >> ~/.bashrc
```
# Install cargo and rust
```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```
# Download and build binary
```sh
git clone https://github.com/near/nearcore 
cd nearcore 
git fetch 
git checkout 0f81dca95a55f975b6e54fe6f311a71792e21698
cargo build -p neard --release --features shardnet
```

# check the version
```sh
~/nearcore/target/release/neard --version
```

# Initialize the working directory, hapus file lama dan unduh file konfigurasi baru dan genesis.json baru. Karena jaringan sulit bercabang baru-baru ini, tidak perlu mengunduh snapshot.
```sh
~/nearcore/target/release/neard --home ~/.near init --chain-id shardnet --download-genesis 
rm ~/.near/config.json ~/.near/genesis.json
```
```sh
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json 
wget -O ~/.near/genesis.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```
```sh
mv ~/.near/data ~/.near/data-fork1 && rm ~/.near/config.json && wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json && rm ~/.near/genesis.json && wget -O ~/.near/genesis.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```

# Sekarang Anda perlu membuat dompet https://wallet.shardnet.near.org/ PERINGATAN! Jika Anda memiliki dompet sebelumnya ke hard fork, Anda mungkin perlu membuatnya kembali! Lupakan mnemonic lama dan lakukan semuanya dari awal.
```sh
near login
```
Lihat tautan otorisasi dan salin ke jendela browser tempat dompet Anda dibuka. Setelah menghubungkan dompet, kanan nama dompet ("accountId".shardnet.near) di CLI dan konfirmasi.

# Salin json dompet Anda dan buat beberapa perubahan
```sh
cd ~/.near-credentials/shardnet/
cp wallet.json ~/.near/validator_key.json
nano validator_key.json
```

# Ubah ID akun dan parameter Private Key yang sesuai, lalu keluar dari editor nano.
```sh
{ 
"account_id": "xx.factory.shardnet.near", 
"public_key":"ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G", "secret_key": "ed25519:****" 
}
```

# Buat file layanan (satu perintah)
```sh
sudo tee /etc/systemd/system/neard.service > /dev/null <<EOF 
[Unit] 
Description=NEARd Daemon Service 
[Service] 
Type=simple 
User=$USER
#Group=near 
WorkingDirectory=$HOME/.near
ExecStart=$HOME/nearcore/target/release/neard run 
Restart=on-failure 
RestartSec=30 
KillSignal=SIGINT 
TimeoutStopSec=45 
KillMode=mixed 
[Install] 
WantedBy=multi-user.target 
EOF
```

# Sekarang mulai layanan dan lihat log, bahwa semuanya berfungsi dengan baik. Anda akan melihatnya mengunduh blok. Menunggu sinkronisasi penuh.
```sh
sudo systemctl daemon-reload 
sudo systemctl enable neard 
sudo systemctl start neard
journalctl -f -u neard
```
```sh
sudo apt install ccze

journalctl -n 100 -f -u neard | ccze -A
```
# Sekarang mari kita gunakan kontrak staking pool kita dengan 30 Near.
```sh
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```
Ubah parameter "pool id", "accountId", "public key", "accountId" di sini!

Contoh:
```sh
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "dinasa", "owner_id": "dinasa.shardnet.near", "stake_public_key": "ed25519:CMJBcTFweJ6CVFSMiR6jRh3ej2SyBp5huG6mHASDefPw", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="dinasa.shardnet.near" --amount=300 --gas=300000000000000
```

# Sekarang kita buat proposal
```sh
near proposals
```

# Ubah parameter untuk mempertaruhkan yang sesuai.
```sh
near call dinasa.factory.shardnet.near  deposit_and_stake --amount 1200 --accountId dinasa.shardnet.near --gas=300000000000000
```
# Dalam beberapa epochs, anda dapat melihat di explorer dengan mengetik:
```sh
near validators current 
near validators next
```
# Panduan Transaksi

Deposit and Stake NEAR
```sh
near call <staking_pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000
```
Unstake NEAR
```sh
near call <staking_pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
Unstake all
```sh
near call <staking_pool_id> unstake_all --accountId <accountId> --gas=300000000000000
```
Withdraw
```sh
near call <staking_pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
Withdraw all
```sh
near call <staking_pool_id> withdraw_all --accountId <accountId> --gas=300000000000000
```
Ping
```sh
near call <staking_pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```
Total Balance
```sh
near view <staking_pool_id> get_account_total_balance '{"account_id": "<accountId>"}'
```
Staked Balance
```sh
near view <staking_pool_id> get_account_staked_balance '{"account_id": "<accountId>"}'
```
Unstaked Balance
```sh
near view <staking_pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'
```
Pause
```sh
near call <staking_pool_id> pause_staking '{}' --accountId <accountId>
```
Resume
```sh
near call <staking_pool_id> resume_staking '{}' --accountId <accountId>
```
Cek Delegators and Stake
```sh
near view <your pool>.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId <accountId>.shardnet.near
```
Cek Reason Validator Kicked
 ```sh
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("<POOL_ID>"))' | jq .reason
```
Cek Blocks
```sh
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.current_validators[] | select(.account_id | contains ("POOL_ID"))'
```
