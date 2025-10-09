<p align="center">
  <img height="100" height="auto" src="https://github.com/user-attachments/assets/d68872f0-006d-4394-bbf0-149d22c73124">
</p>

# Arcium Testnet

Official documentation:
>- [Validator setup instructions](https://docs.arcium.com/developers/node-setup)

Explorer:
>- [Explorer](-)

### Minimum Hardware Requirements
 - 12x CPUs; the faster clock speed the better
 - 32GB RAM
 - 100GB of storage (SSD or NVME)

### Recommended Hardware Requirements 
 - 24x CPUs; the faster clock speed the better
 - 64GB RAM
 - 100GB+ of storage (SSD or NVME)

 - Ubuntu 24.04

Устанавливаем ноду:

```
sudo apt update & sudo apt upgrade -y
```

```
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev git-all protobuf-compiler screen libgbm1 ca-certificates gnupg -y
```

```
curl -fsSL https://get.docker.com | sh
```
```
sudo systemctl enable docker
```
```
sudo systemctl restart docker
```
```
sudo usermod -aG docker $USER
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Выбираем 1 - Enter

``source $HOME/.cargo/env``

```
mkdir arcium-node-setup
cd arcium-node-setup
```

```
curl --proto '=https' --tlsv1.2 -sSfL https://arcium-install.arcium.workers.dev/ | bash
```

```
PATH="/root/.local/share/solana/install/active_release/bin:$PATH"
```

```
apt install cmdtest
```

```
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw allow 8080
sudo ufw enable
```

```
curl --proto '=https' --tlsv1.2 -sSfL https://arcium-install.arcium.workers.dev/ | bash
```

Создаем 3 вида ключей. Обязательно сохраните файлы: node-keypair.json, callback-kp.json, identity.pem

1) Сохраняем pubkey и seed phrase:

```
solana-keygen new --outfile node-keypair.json --no-bip39-passphrase
```

<img width="806" height="215" alt="Termius_iEX8SUd1on" src="https://github.com/user-attachments/assets/c20a4505-356d-4bd7-b8b5-19291c159865" />

2) Сохраняем pubkey и seed phrase:

```
solana-keygen new --outfile callback-kp.json --no-bip39-passphrase
```

<img width="781" height="182" alt="Termius_T1GaFMcN4d" src="https://github.com/user-attachments/assets/ad9a9126-0a50-4f5b-aef1-2ffc75c4b8cf" />

3) Ключ будет хранится в пути $HOME/arcium-node-setup/identity.pem
   Сохраните его.

```
openssl genpkey -algorithm Ed25519 -out identity.pem
```

Теперь нужно пополнить 2 адреса devnet $SOL. Сначала получим их адреса.

```
solana address --keypair node-keypair.json
```

```
solana address --keypair callback-kp.json
```

Пополняем кошельки devnet $SOL. В первом случае меняем <node-pubkey> на свой адрес. Пример: solana airdrop 2 6NKm6jaQrvE5NPEdwSpGXDUfvZs8huS7gpCVRPc2o8wp -u devnet

```
solana airdrop 2 <node-pubkey> -u devnet
```

Во втором случае меняем <callback-pubkey> на свой адрес. Пример: solana airdrop 2 CjiZdHGqPq3Ve3NFPVRD4LvFrqAg4tkAdjUejsNALxdf -u devnet

```
solana airdrop 2 <callback-pubkey> -u devnet
```

Если кошельки не пополнились, значит пробуем использовать кран на сайте - https://faucet.solana.com/

```
solana config set --url https://api.devnet.solana.com
```

Меняем --node-offset и --ip-address:

--node-offset: Представьте себе уникальный идентификатор вашего узла в сети. Выберите большое случайное число (например, 8–10 цифр), чтобы оно не конфликтовало с другими узлами. Если во время настройки возникнет ошибка с сообщением о том, что ваш номер уже занят, просто выберите другой и повторите попытку.

--ip-address: Публичный IP-адрес вашего узла

```
arcium init-arx-accs \
  --keypair-path node-keypair.json \
  --callback-keypair-path callback-kp.json \
  --peer-keypair-path identity.pem \
  --node-offset <your-node-offset> \
  --ip-address <your-node-ip> \
  --rpc-url https://api.devnet.solana.com
```

Добавляем конфиг:

[node]
offset = <your-node-offset>  # Ваши придуманные цифры из шага выше
hardware_claim = 0
starting_epoch = 0
ending_epoch = 9223372036854775807

[network]
address = "0.0.0.0"

[solana]
endpoint_rpc = "<your-rpc-provider-url-here>"  # Замените URL-адресом вашего RPC-провайдера или используйте https://api.devnet.solana.com по умолчанию.
endpoint_wss = "<your-rpc-websocket-url-here>"   # Замените URL-адресом WebSocket вашего RPC-провайдера или используйте wss://api.devnet.solana.com по умолчанию.
cluster = "Devnet"
commitment.commitment = "confirmed"

```
nano node-config.toml
```

Операции кластера
Кластеры — это группы узлов, которые совместно выполняют вычисления во время стресс-тестирования. Вы можете создать свой собственный кластер и пригласить других присоединиться к нему или присоединиться к существующему кластеру, созданному кем-то другим.

Кластеры — это группы узлов, которые совместно выполняют вычисления MPC во время стресс-тестирования. Существует два варианта участия в кластере.

Вариант 1: Создание собственного кластера

```
arcium init-cluster \
  --keypair-path node-keypair.json \
  --offset <cluster-offset> \
  --max-nodes <max-nodes> \
  --rpc-url https://api.devnet.solana.com
```

Параметры:

--offset: Уникальный идентификатор вашего кластера (отличается от узла) (например, 8–10 цифр)

--max-nodes: Максимальное количество узлов в кластере, до 10

Вариант 2: Присоединиться к существующему кластеру
Чтобы присоединиться к существующему кластеру, вам необходимо получить приглашение от администрации кластера. После получения приглашения примите его:

```
arcium join-cluster true \
  --keypair-path node-keypair.json \
  --node-offset <your-node-offset> \
  --cluster-offset <cluster-offset> \
  --rpc-url https://api.devnet.solana.com
```

Параметры:

true: Принять запрос на присоединение (false — отклонить)

--node-offset: Смещение вашего узла

--cluster-offset: Номер кластера, к которому вы присоединяетесь

Запускаем ноду:

```
mkdir -p arx-node-logs && touch arx-node-logs/arx.log
```

```
docker run -d \
  --name arx-node \
  -e NODE_IDENTITY_FILE=/usr/arx-node/node-keys/node_identity.pem \
  -e NODE_KEYPAIR_FILE=/usr/arx-node/node-keys/node_keypair.json \
  -e OPERATOR_KEYPAIR_FILE=/usr/arx-node/node-keys/operator_keypair.json \
  -e CALLBACK_AUTHORITY_KEYPAIR_FILE=/usr/arx-node/node-keys/callback_authority_keypair.json \
  -e NODE_CONFIG_PATH=/usr/arx-node/arx/node_config.toml \
  -v "$(pwd)/node-config.toml:/usr/arx-node/arx/node_config.toml" \
  -v "$(pwd)/node-keypair.json:/usr/arx-node/node-keys/node_keypair.json:ro" \
  -v "$(pwd)/node-keypair.json:/usr/arx-node/node-keys/operator_keypair.json:ro" \
  -v "$(pwd)/callback-kp.json:/usr/arx-node/node-keys/callback_authority_keypair.json:ro" \
  -v "$(pwd)/identity.pem:/usr/arx-node/node-keys/node_identity.pem:ro" \
  -v "$(pwd)/arx-node-logs:/usr/arx-node/logs" \
  -p 8080:8080 \
  arcium/arx-node
```

Полезные команды:

Cтатус узла:

```
arcium arx-info <your-node-offset> --rpc-url https://api.devnet.solana.com
```

Проверить активна ли нода:

```
arcium arx-active <your-node-offset> --rpc-url https://api.devnet.solana.com
```

Посмотреть логи:

```
docker logs -f arx-node
```


