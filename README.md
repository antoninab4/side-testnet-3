# side-testnet-3

## Характеристики сервера

Minimum hardware requirements:
Testnet	4	8GB	150GB

## 1) проверяем версию
```
sided version --long | grep -e commit -e version
```
вывод должен быть такой

#version: 0.7.0-rc2
#commit: abc51da52f8a612e2bbb25ca763b87815b0ba060

## 2) проверяем кошелек и имя кошелька
```
sided keys list
```
пример вывода 


- address: bc1zw58dtwzs9kawemhdw0h52zsv2ftt6k77734xs

  name: wingsnodeteam

  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Al1y71RByeAuhzzIJR74sd709O1Cx0pXlsD0MxQBSa"}'

  type: local

## 3) добавляем в cli

```
cd side
```
```
sided config chain-id side-testnet-3
```
## 4) Download Genesis
```
wget https://github.com/sideprotocol/testnet/raw/main/side-testnet-3/genesis.json -O ~/.side/config/genesis.json
```
## 5) проверка генезиса 
```
shasum -a 256 ~/.side/config/genesis.json
```
 вывод типа 2e908a79fee2c70c93b41eba3842106f8370b1cf  genesis.json

## 6) Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uside\"/;" ~/.side/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.side/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.side/config/config.toml
seeds="00170c0c23c3e97c740680a7f881511faf68289a@202.182.119.24:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.side/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.side/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.side/config/config.toml
```

## 7) pruning
```
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.side/config/app.toml
```
## 8) Indexer (optional)
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.side/config/config.toml
```
## 9) Download addrbook
```
wget -O $HOME/.side/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Side_Protocol/addrbook.json"
```
## 10) Create a service file
```
sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
[Unit]
Description=sided
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## 10.1) меняем порты на нестандартные (если не одна нода на сервере)
```
sed -i.bak -e "s%:26658%:34658%; s%:26657%:34657%; s%:6060%:6860%; s%:26656%:34656%; s%:26660%:34660%" $HOME/.side/config/config.toml && sed -i.bak -e "s%:9090%:9890%; s%:9091%:9891%; s%:1317%:2117%; s%:8545%:9345%; s%:8546%:9346%; s%:6065%:6865%" $HOME/.side/config/app.toml && sed -i.bak -e "s%:26657%:34657%" $HOME/.side/config/client.toml 
```

## 11) стартуем и ждем запуска сети
```
sudo systemctl daemon-reload
sudo systemctl enable sided
sudo systemctl restart sided && sudo journalctl -u sided -f -o cat
```


## Полезные команды

- проверить баланс
```
sided query bank balances sided...addressjkl1yjgn7z09ua9vms259j
```
- Info
```
sided status 2>&1 | jq .NodeInfo
```
```
sided status 2>&1 | jq .SyncInfo
```
```
sided status 2>&1 | jq .ValidatorInfo
```
- Check node logs
```
sudo journalctl -fu sided -o cat
```
- Check service status
```
sudo systemctl status sided
```
- Restart service
```
sudo systemctl restart sided
```
- Stop service
```
sudo systemctl stop sided
```
- Start service
```
sudo systemctl start sided
```
- reload/disable/enable
```
sudo systemctl daemon-reload
```
```
sudo systemctl disable sided
```
```
sudo systemctl enable sided
```

- Your Peer
```
echo $(sided tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.side/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```
- создать / восстановить кошелек

New Key or Recover Key
```
sided keys add Wallet_Name
````
     OR
```
sided keys add Wallet_Name --recover
```

- Check all keys
```
sided keys list
```
- Check Balance
```
sided query bank balances sided...addressjkl1yjgn7z09ua9vms259j
```
- Delete Key
```
sided keys delete Wallet_Name
```
 Validator Management Edit Validator (изменить данные валидатора)

```
sided tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "id_картинки" \
--details "@WingsNodeTeam" \
--website "telegram @WingsNodeTeam" \
--security-contact "WingsNodeTeam@gmail.com" \
--chain-id side-testnet-3 \
--commission-rate 0.05 \
--from ваш_кошелек \
--gas 350000 -y
```

Your Valoper-Address
```
sided keys show Wallet_Name --bech val
```

- Jail Info (проверить на тюрьму)
```
sided query slashing signing-info $(sided tendermint show-validator)
```
- Unjail (выйти тз тюрьмы)
```
sided tx slashing unjail --from Wallet_name --chain-id side-testnet-3 --gas 350000 -y
```

- Withdraw all rewards from all validators
```
sided tx distribution withdraw-all-rewards --from Wallet_Name --chain-id side-testnet-3 --gas 350000 -y
```
- Withdraw and commission from your Validator
```
sided tx distribution withdraw-rewards sidedvaloper1a........ --from Wallet_Name --gas 350000 --chain-id=side-testnet-3 --commission -y
```
- Delegate tokens to your validator
```
sided tx staking delegate Your_sidedvalpoer........ "100000000"uside --from Wallet_Name --gas 350000 --chain-id=side-testnet-3 -y
```
- Delegate tokens to different validator
```
sided tx staking delegate sidedvalpoer........ "100000000"uside --from Wallet_Name --gas 350000 --chain-id=side-testnet-3 -y
```
- Transfer tokens from wallet to wallet
```
sided tx bank send Your_sidedaddress............ sidedaddress........... "1000000000000000000"uside --gas 350000 --chain-id=side-testnet-2 -y
```


- Удалить ноду 
```
sudo systemctl stop sided
sudo systemctl disable sided
rm /etc/systemd/system/sided.service
sudo systemctl daemon-reload
cd $HOME
rm -rf side
rm -rf .side
rm -rf $(which sided)
```
