# Create a complete 3-tiered microservice with WasmEdge

## What you will learn

* The way to develop wasm-based microservices
* How to use WasmEdge to run microservice with MySQL

## Configure your environment

To finish this tutorial, you will need to instal the following tools. To make life happy, using a container to install them will be easier :-)

1. [Install Rust Toolchain](https://www.rust-lang.org/tools/install)
2. [Install WasmEdge](https://wasmedge.org/book/en/quick_start/install.html)
3. [Install and Start the MySQL Database](https://dev.mysql.com/doc/mysql-installation-excerpt/8.0/en/)

#### Installation Guide On Linux

```
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
# Install WebAssembly target for Rust
rustup target add wasm32-wasi

# Install WasmEdge
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -e all
source $HOME/.wasmedge/env

# Install MySQL
sudo apt install mysql-server
sudo systemctl start mysql.service
mysql_secure_installation # Configure your MySQL server
```

## Get the source code

If you already cloned in tutorial 1, then just use it!

Git clone the [second-state/microservice-rust-mysql](https://github.com/second-state/microservice-rust-mysql) template project. 

```
git clone https://github.com/second-state/microservice-rust-mysql.git
```

This is a database-driven web application with one WasmEdge "container" for the entire web service (microservice), and two Linux containers for supporting services — one for the MySQL database and one for the NGINX  to serve the static HTML page for the frontend UI.

## The workflow

1. Build the application with Rust.
2. Compile the Wasm with WasmEdge Ahead-of-Time Compiler(wasmedgec) for the better performance.
3. Run the application with WasmEdge
4. Play with it

## Build this application

Open the cloned microservice-rust-mysql project on your terminal.

```
cd microservice-rust-mysql
```

Then, use `cargo` to build the application. A wasm file will be generated.

```
cargo build --target wasm32-wasi --release
# The wasm file will be here: target/wasm32-wasi/release/order_demo_service.wasm
```

## Improve the performance with the AOT compiler

You can run the AOT compiler on the wasm file. It could significantly improvement the performance of compute-intensive applications. This microservice, however, is a network intensitive application. Our use of async HTTP networking (Tokio and hyper) and async MySQL connectors are crucial for the performance of this microservice.

```
wasmedgec target/wasm32-wasi/release/order_demo_service.wasm order_demo_service.wasm
```

## Run the application

You can use the wasmedge command to run the wasm application. It will start the server. Make sure that you pass the MySQL connection string as the env variable to the command.

```
wasmedge --env "DATABASE_URL=mysql://user:passwd@127.0.0.1:3306/mysql" order_demo_service.wasm
```

## CRUD tests

Open another terminal, and you can use the `curl` command to interact with the web service.

When the microservice receives a GET request to the `/init` endpoint, it would initialize the database with the `orders` table.

```bash
curl http://localhost:8080/init
```

When the microservice receives a POST request to the `/create_order` endpoint, it would extract the JSON data from the POST body and insert an `Order` record into the database table.
For multiple records, use the `/create_orders` endpoint and POST a JSON array of `Order` objects.

```bash
curl http://localhost:8080/create_orders -X POST -d @orders.json
```

When the microservice receives a GET request to the `/orders` endpoint, it would get all rows from the `orders` table and return the result set in a JSON array in the HTTP response.

```bash
curl http://localhost:8080/orders
```

When the microservice receives a POST request to the `/update_order` endpoint, it would extract the JSON data from the POST body and update the `Order` record in the database table that matches the `order_id` in the input data.

```bash
curl http://localhost:8080/update_order -X POST -d @update_order.json
```

When the microservice receives a GET request to the `/delete_order` endpoint, it would delete the row in the `orders` table that matches the `id` GET parameter.

```bash
curl http://localhost:8080/delete_order?id=2
```

## Use Web UI

Go to `client` folder. Run the following python command for hosting a simple HTTP server:

```
python3 -m http.server 9000
```

Open your browser with `127.0.0.1:9000`. Then, you can find a simple WebSite which is using the deployed microservice.

That's it.
