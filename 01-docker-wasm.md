# Create a complete 3-tiered microservice with Docker+Wasm

## What you will learn

* The way to develop wasm-based microservices
* How to use Docker+Wasm to run Wasm containers side by side with Linux containers

## Configure your environment

You only need to download the [Docker Desktop](https://docs.docker.com/desktop/install/). The version of Docker Desktop should be at least 4.15.

## Get the source code

Git clone the [second-state/microservice-rust-mysql](https://github.com/second-state/microservice-rust-mysql) template project. 

```
git clone https://github.com/second-state/microservice-rust-mysql.git
```

This is a database-driven web application with one WasmEdge "container" for the entire web service (microservice), and two Linux containers for supporting services — one for the MySQL database and one for the NGINX  to serve the static HTML page for the frontend UI. The entire microservice application source code is here.

https://github.com/second-state/microservice-rust-mysql/blob/main/src/main.rs

<img width="819" alt="image" src="https://user-images.githubusercontent.com/45785633/233396438-378d029e-15e2-4fc9-9dd8-83dd1bd917c6.png">



## Build, share, and run this application

Open the cloned microservice-rust-mysql project on your terminal.

````
cd microservice-rust-mysql
````

Then, use docker compose to build, share, and run the application

```
docker compose up
```

## Test

Open the URL http://localhost:8090 in a browser and create a sample order to test it.


## Next

Next, let's deep dive into the 3-tiered microservice.





