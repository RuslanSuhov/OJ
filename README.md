# Установка ноды OJO

## Технические требования

4CPU 8RAM 100GB

## Дискорд проекта
https://discord.gg/ptpt3tt4NB
### Вступаем в дискорд, проходим все ветки, все как обычно.

## Подготовка сервера

# Подготовка сервера
```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install make clang pkg-config libssl-dev build-essential git gcc chrony curl jq ncdu bsdmainutils htop net-tools lsof fail2ban wget -y
```
# install dependencies, if needed

```
sudo apt install -y curl git jq lz4 build-essential unzip
```

```
bash <(curl -s "https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/go_install.sh")
```
```
source .bash_profile
```

## Важно!!! Копируем себе, вводим (придумываеи имя ноды) и вставляем в терминал
```
NODE_MONIKER="Здесь_Ваше_имя_ноды_английскими"
```
### далее
```
cd || return
```
```
rm -rf ojo
```
```
git clone https://github.com/ojo-network/ojo
```
```
cd ojo || return
```
```
git checkout v0.1.2
```
```
make install
```
```
ojod version # HEAD-ad5a2377134aa13d7d76575b95613cf8ed12d1e4
```
```
ojod config keyring-backend test
```
```
ojod config chain-id ojo-devnet
```
```
ojod init "$NODE_MONIKER" --chain-id ojo-devnet
```
```
curl -Ls https://rpc.devnet-n0.ojo-devnet.node.ojo.network/genesis > $HOME/.ojo/config/genesis.json
```
```
curl -s https://snapshots2-testnet.nodejumper.io/ojo-testnet/addrbook.json > $HOME/.ojo/config/addrbook.json
```
```
SEEDS=""
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.ojo/config/config.toml
```
```
sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.ojo/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.ojo/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.ojo/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.ojo/config/app.toml
```
```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uojo"|g' $HOME/.ojo/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.ojo/config/config.toml
```
```
sudo tee /etc/systemd/system/ojod.service > /dev/null << EOF
[Unit]
Description=Ojo Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which ojod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```
```
ojod tendermint unsafe-reset-all --home $HOME/.ojo --keep-addr-book
```
```
SNAP_NAME=$(curl -s https://snapshots2-testnet.nodejumper.io/ojo-testnet/info.json | jq -r .fileName)
curl "https://snapshots2-testnet.nodejumper.io/ojo-testnet/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C $HOME/.ojo
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable ojod
```
```
sudo systemctl start ojod
```
```
sudo journalctl -u ojod -f --no-hostname -o cat
```

### Ждем синхронизации
## Актуальную высоту ноды смотрим здесь 
https://ojo.explorers.guru/blocks

## Теперь нам надо дождаться пока нода синхронизируется с сетью (высотав логах ноды будет такая же как и в эксплоере)



## создаем кошелек для ноды
```
ojod keys add wallet
```

## Вывод будет такой при создании кошелька
## - name: wallet
##  type: local
##  address: ojo1ela8c0jhqgjsj2cq7twu9uhda2n8e6cs8ztxs3
##  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Auq9WzVEs5pCoZgr2WctjI7fU+lJCH0I3r6GC1oa0tc0"}'
##  mnemonic: ""

## !!!Записываем сид фразу !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Копируем и сохраняем, иначе будет потом не возможно восстановить ноду
## Ниже пример сид фразы
### kite upset hip dirt pet winter thunder slice parent flag sand express suffer chest custom pencil mother bargain remember patient other curve cancel sweet


## Еще способ проверит, синхронизировалась ли нода. Если False то да
ojod status 2>&1 | jq .SyncInfo.catching_up

## Запросить токены сейчас нельзя, они нужны для создания валидатора, но вы скидываете мне адрес кошелька своего от ноды и я вам присылаю токены для валидатора 10


## Проверяем баланс
ojod q bank balances $(ojod keys show wallet -a)

## вывод примерно такой
###  balances:
###  - amount: "10000000"
###    denom: uojo

### Создаем валидатора (где написано --amount=9000000uojo) вместо 9000000 вписываете свою сумму, но не всю что на кошельке, оставьте на комиссии, иначе встрянете

```
ojod tx staking create-validator \
--amount=9000000uojo \
--pubkey=$(ojod tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=ojo-devnet \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--fees=2000uojo \
--from=wallet \
-y
```


## make sure you see the validator details
```
ojod q staking validator $(ojod keys show wallet --bech val -a)
```

# install pricefeeder
```
cd || return
rm -rf price-feeder
git clone https://github.com/ojo-network/price-feeder
cd price-feeder || return
git checkout v0.1.1
make install
price-feeder version # version: HEAD-5d46ed438d33d7904c0d947ebc6a3dd48ce0de59
```
```
mkdir -p $HOME/.price-feeder
curl -s https://raw.githubusercontent.com/ojo-network/price-feeder/main/price-feeder.example.toml > $HOME/.price-feeder/price-feeder.toml
```
```
ojod keys add feeder-wallet --keyring-backend os
ojod tx bank send wallet YOUR_FEEDER_ADDRESS 10000000uojo --from wallet --chain-id ojo-devnet --fees 2000uojo -y
ojod q bank balances $(ojod keys show feeder-wallet --keyring-backend os -a)
```
```
CHAIN_ID=ojo-devnet
```
### ниже строку копируем себе и вводим ваш пароль от ноды и вставляем измененный в терминал
```
KEYRING_PASSWORD=YOUR_KEYRING_PASSWORD
```
```
WALLET_ADDRESS=$(ojod keys show wallet -a)
FEEDER_ADDRESS=$(ojod keys show feeder-wallet --keyring-backend os -a)
VALIDATOR_ADDRESS=$(ojod keys show wallet --bech val -a)
GRPC="localhost:9090"
RPC="http://localhost:26657"
```
```
ojod tx oracle delegate-feed-consent $WALLET_ADDRESS $FEEDER_ADDRESS --from wallet --fees 2000uojo -y
ojod q oracle feeder-delegation $VALIDATOR_ADDRESS
```
```
sed -i '/^dir *=.*/a pass = ""' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^address *=.*|address = "'$FEEDER_ADDRESS'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^chain_id *=.*|chain_id = "'$CHAIN_ID'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^validator *=.*|validator = "'$VALIDATOR_ADDRESS'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^backend *=.*|backend = "os"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^dir *=.*|dir = "'$HOME/.ojo'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^pass *=.*|pass = "'$KEYRING_PASSWORD'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^grpc_endpoint *=.*|grpc_endpoint = "'$GRPC'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^tmrpc_endpoint *=.*|tmrpc_endpoint = "'$RPC'"|g' $HOME/.price-feeder/price-feeder.toml
sed -i 's|^global-labels *=.*|global-labels = [["chain_id", "'$CHAIN_ID'"]]|g' $HOME/.price-feeder/price-feeder.toml
```
```
sudo tee /etc/systemd/system/price-feeder.service > /dev/null << EOF
[Unit]
Description=Ojo Price Feeder
After=network-online.target
[Service]
User=$USER
ExecStart=$(which price-feeder) $HOME/.price-feeder/price-feeder.toml --log-level debug
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
Environment="PRICE_FEEDER_PASS=$KEYRING_PASSWORD"
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable price-feeder
sudo systemctl start price-feeder
```
```
sudo journalctl -u price-feeder -f --no-hostname -o cat
```
