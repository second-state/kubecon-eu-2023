# Do more with Dapr

## What you will learn

* The Dapr sidecar makes over 100 common infrastructure services available to its connected microservice apps.
* It is a perfect fit for Wasm microservices where drivers and client libraries are lacking.
* WasmEdge provides a Rust SDK to connect to the Dapr sidecar from WasmEdge.

## Setup your environment

1. Install WasmEdge with its AI inference extensions.

```
VERSION=0.11.1
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | sudo bash -s -- -e all --version=$VERSION --tf-version=$VERSION --tf-deps-version=$VERSION --tf-tools-version=$VERSION -p /usr/local
```

2. Install Rust and its Wasm compiler target

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32-wasi
```

3. Install and init the latest Dapr sidecar

```
wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash
dapr init
```

4. Start the Dapr sentry service

```
mkdir -p .dapr/certs
wget https://github.com/dapr/dapr/releases/download/v1.8.4/sentry_linux_amd64.tar.gz
tar -zxvf sentry_linux_amd64.tar.gz
nohup ./sentry --issuer-credentials .dapr/certs --trust-domain cluster.local &
```

5. Install and start MySQL

```
sudo apt-get -y install mysql-server libmysqlclient-dev curl
sudo service mysql start

mysql -e "SET GLOBAL max_allowed_packet = 36700160;" -uroot -proot
mysql -e "SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY = WARN;" -uroot -proot
mysql -e "SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY = ON;" -uroot -proot
mysql -e "SET @@GLOBAL.GTID_MODE = OFF_PERMISSIVE;" -uroot -proot
mysql -e "SET @@GLOBAL.GTID_MODE = ON_PERMISSIVE;" -uroot -proot
mysql -e "SET @@GLOBAL.GTID_MODE = ON;" -uroot -proot
mysql -e "PURGE BINARY LOGS BEFORE now();" -uroot -proot
```

## Get the source code

Git clone the [second-state/dapr-wasm](https://github.com/second-state/dapr-wasm) template project. 

```
git clone https://github.com/second-state/dapr-wasm
```

## Dapr features used

* Service invocation
* Secrets store
* Key value store

![](https://github.com/second-state/dapr-wasm/blob/main/docs/dapr-wasmedge.png)

## Build and run

Microservice #1: Turn an image into grayscale

```
cd image-api-grayscale
cargo build --target wasm32-wasi --release
wasmedgec target/wasm32-wasi/release/image-api-grayscale.wasm image-api-grayscale.wasm
nohup dapr run --app-id image-api-grayscale --app-protocol http --app-port 9005 --dapr-http-port 3503 --components-path ../config --log-level debug wasmedge image-api-grayscale.wasm > server.log 2>&1 &
```

Microservice #2: Image classification

```
cd image-api-classify
cargo build --target wasm32-wasi --release
wasmedgec target/wasm32-wasi/release/wasmedge_hyper_server_tflite.wasm wasmedge_hyper_server_tflite.wasm
nohup dapr run --app-id image-api-classify --app-protocol http --app-port 9006 --dapr-http-port 3504 --components-path ../config --log-level debug wasmedge-tensorflow-lite wasmedge_hyper_server_tflite.wasm > server.log 2>&1 &
```

Microservice #3: Events storage and reporting

```
cd events-service
cargo build --target wasm32-wasi --release
wasmedgec target/wasm32-wasi/release/events_service.wasm events_service.wasm
nohup dapr run --app-id events-service --app-protocol http --app-port 9007 --dapr-http-port 3505 --components-path ../config --log-level debug wasmedge events_service.wasm > server.log 2>&1 &
```

## Test

Init the events database from microservice #3.

```
curl http://localhost:9007/init
{"status":true}

curl http://localhost:9007/events
[]
```

Call microservice #2 to turn an image into grayscale.

```
curl -o food-bw.jpg http://localhost:9005/grayscale -X POST --data-binary '@food.jpg'
```

Call microservice #3 to classify an image.

```
curl http://localhost:9006/classify -X POST --data-binary '@food.jpg'
hotdog is detected with 255/255 confidence
```

Query the events database from microservice #3.

```
curl http://localhost:9007/events
[{"id":1,"event_ts":1682029073500,"op_type":"grayscale","input_size":68016},{"id":2,"event_ts":1682029073663,"op_type":"classify","input_size":68016}]
```

