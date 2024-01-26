### Linkler
 * [Hercules Telegram](https://t.me/HerculesNode)
 * [Hercules Twitter](https://twitter.com/Herculesnode)

## 🟢 Sistem özellikleri

Önerilern:
- 16 RAM
- 1 TB disk


## 🟢 Sistem Güncelleme
```shell
sudo apt update & sudo apt upgrade
```

## 🟢 Docker Setup

```shell
sudo apt install docker-compose
```

```shell

sudo apt-get update && sudo apt install jq && sudo apt install apt-transport-https ca-certificates curl software-properties-common -y && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" && sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin && sudo apt-get install docker-compose-plugin

```


## 🟢 Gerekli Klasör ve kurulum dosyası oluşturma

```shell
mkdir sub_data
```

run_worker.sh dosyası oluşturalım.

```shell
nano run_worker.sh
```

bu kodu run_worker.sh dosyasının içine yapıştırın CTRL + X + YES yapıp kaydedelim.

```shell
#!/usr/bin/env sh

docker compose --version >/dev/null || (echo "Docker compose not installed"; exit 1)

if [ ! -d "$1" ]
then
  echo "Provided data directory ($1) does not exist. Usage: $0 <DATA_DIR> <DOCKER_COMPOSE_ARGS>"
  exit 1
fi

# Get absolute path
DATA_DIR="$(cd "$(dirname -- "$1")" >/dev/null; pwd -P)/$(basename -- "$1")"
echo "Using data dir $DATA_DIR"
shift

export USER_ID=$(id -u)
export GROUP_ID=$(id -g)

cat <<EOF > docker-compose.yml
version: "3.8"

services:

  rpc_node:
    image: subsquid/rpc-node:0.2.1
    environment:
      P2P_LISTEN_ADDR: /ip4/0.0.0.0/tcp/${LISTEN_PORT:-12345}
      P2P_PUBLIC_ADDRS: ${PUBLIC_ADDR}
      RPC_LISTEN_ADDR: 0.0.0.0:50051
      BOOT_NODES: >
        12D3KooWSRvKpvNbsrGbLXGFZV7GYdcrYNh4W2nipwHHMYikzV58 /dns4/testnet.subsquid.io/tcp/22345,
        12D3KooWQC9tPzj2ShLn39RFHS5SGbvbP2pEd7bJ61kSW2LwxGSB /dns4/testnet.subsquid.io/tcp/22346
      KEY_PATH: /app/data/key
    volumes:
      - ./:/app/data
    user: "${USER_ID}:${GROUP_ID}"
    ports:
      - "${LISTEN_PORT:-12345}:${LISTEN_PORT:-12345}"

  worker:
    depends_on:
      rpc_node:
        condition: service_healthy
    image: subsquid/p2p-worker:0.2.0
    environment:
      PROXY_ADDR: rpc_node:50051
      SCHEDULER_ID: 12D3KooWQER7HEpwsvqSzqzaiV36d3Bn6DZrnwEunnzS76pgZkMU
      LOGS_COLLECTOR_ID: 12D3KooWC3GvQVqnvPwWz23sTW8G8HVokMnox62A7mnL9wwaSujk
      AWS_ACCESS_KEY_ID: 66dfc7705583f6fd9520947ac10d7e9f
      AWS_SECRET_ACCESS_KEY: a68fdd7253232e30720a4c125f35a81bd495664a154b1643b5f5d4a4a5280a4f
      AWS_S3_ENDPOINT: https://7a28e49ec5f4a60c66f216392792ac38.r2.cloudflarestorage.com
      AWS_REGION: auto
      SENTRY_DSN: https://3d427b41736042ae85010ec2dc864f05@o1149243.ingest.sentry.io/4505589334081536
      RPC_URL: ${RPC_URL:-https://sepolia-rollup.arbitrum.io/rpc}
      MAX_GET_LOG_BLOCKS: ${MAX_GET_LOG_BLOCKS:-50000}
    volumes:
      - ${DATA_DIR}:/app/data
    ports:
      - "${PROMETHEUS_PORT:-9090}:9090"
    user: "${USER_ID}:${GROUP_ID}"
    deploy:
      resources:
        limits:
          memory: 16G
EOF

exec docker compose "$@"
```


![image](https://github.com/HerculesNode/subsquid/assets/101635385/f6e5c8d1-4c59-445d-ba73-491f87983b83)

## 🟢 izin verelim ve PEER ID ALALIM.

```shell
chmod +x run_worker.sh
```

Aşağıdaki komut PEER ID vereccek bunu kaydedin. Ayrıca key adında bir dosya oluşturacak bunuda yedekleyin.

```
docker run --rm subsquid/rpc-node:0.2.1 keygen >key
```
![image](https://github.com/HerculesNode/subsquid/assets/101635385/7ae731d7-3de9-41d9-9c38-a1a7c726210a)



diğer detaylar başlayınca yazılacak. Şimdilik kurulum yapmayın.
