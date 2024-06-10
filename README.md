# ZEB Network

## Overview

Zeb network is a p2p relay network for verifying vlc clock causal order. This system is currently in poc stage, and contains several components p2p module, vlc module, gateway module, browser, chat client.

## Implementations

Follow the latest poc implementations

- p2p: [znet](https://github.com/bufrr/znet)
- zchronod: [chronos](https://github.com/NagaraTech/chronos)
- [gateway](https://github.com/NagaraTech/gateway)
- Pigeon: [z-message-extension](https://github.com/NagaraTech/z-message-extension)
- Dolphin: [z-browser](https://github.com/NagaraTech/z-browser)
- [p2p-docker](https://github.com/NagaraTech/p2p-docker)

## **znet**

ZNet is a p2p relay network with verifiable VLC (virtual logic clock) causal order. This system is currently in poc stage.

### **Install**

#### **build from source**

```shell
$ git clone https://github.com/bufrr/znet.git
$ cd znet
$ make
$ ./build/znet -h

# Run node with create mode: 
$ go run main.go --id Hello0 --domain 127.0.0.1
# Run node with join mode: 
$ go run main.go --remote tcp://127.0.0.1:33333 --id Hello0 --p2p 33334 --ws 23334
```

### How to test

```
go run examples/bin/basic.go
```

### Benchmark

```
cd example
go test -bench='^\QBenchmarkStartCluster\E$'
```

## zchronod

The zchronod or chronos is a implement of vlc(verifiable logical clock).

It use the znet p2p relay as network module. And as a backend project node for supporting verifiable logical clock and causality ordering. This system is currently in poc stage.

### Dependences

#### PostgreDB

The zchronod depends on postgre db for data persistence, so please install postgre and setup a pg instance.

#### Znet p2p relayer

The zchronod play a role of inner logic and state layer in vlc overview. One zchronod process matches a znet p2p node, and them use inner net socket for communication.

For now, zchronod and znet use the same node identity for two processes. So first generate a key pair identity, then address it to `node_id` in [config-tempelete.yaml](https://github.com/NagaraTech/chronos/blob/feat/vlc-inner-node/zchronod/config-tempelete.yaml) of zchronod.

#### Net messaging

The zchronod and znet using protobuf proto3 as serialization compression algorithm and communication protocol. More messages body details, please see [crates/protos](https://github.com/NagaraTech/chronos/blob/feat/vlc-inner-node/crates/protos) for check it.

### Compile

#### Build from source

```
git clone https://github.com/NagaraTech/chronos.git

cd chronos

cargo build -p zchronod
```

### Run a node

```
# 1.for help info
./target/debug/zchronod -h

# 2.init db & dna business pg tables
./target/debug/zchronod --init_pg postgres://postgres:hetu@0.0.0.0:5432/vlc_inner_db

# 3.setup the node & please update the config file to your dev environment
./target/debug/zchronod --config ./zchronod/config-tempelete.yaml
```

### How to test

```
cargo run --package zchronod --bin client_write
cargo run --package zchronod --bin client_read
```

## Gateway

The gateway is for collecting P2P VLC date periodically and providing APIs for the browser and Chrome extension to query relative data.

### Dependences

#### PostgreDB

The gateway depends on postgre db for data persistence, so please install postgre and setup a pg instance.

#### Environment Variables Config

The .env file contains the database configuration and http restful server port and seed node infos. Please config it before you deploy the gateway with your requirements, for more details please refer [.env](https://github.com/NagaraTech/gateway/blob/master/.env).

### Seed Node

With seed node, gateway can use bfs_traverse to acquire the whole P2P network nodes info and data from VLC.

### Restful APIs

Gateway provides restful apis:

- /gateway
  - /overview (provide P2P network nodes brief infos. ex. node ids)
  - /node/:id (provide single node detailed info. ex. is-alive,clock,message ids )
  - /message/:id (provide single message details info.)
  - /merge_log_by_message_id/:id (provide the relative merge logs of the message)

#### Acquire vlc data from P2P

The gateway uses protobuf proto3 as serialization compression algorithm and communication protocol for querying from P2P, More messages body details, please see [src/proto](https://github.com/NagaraTech/gateway/blob/master/src/proto) for check it.

### Compile

#### Build from source

```
git clone https://github.com/NagaraTech/gateway.git
cargo build
```

### Run a node

```
# 1.init db & pg tables
./target/debug/migration
# 2.setup gateway
./target/debug/restful-server 
```

### How to test

Python sctipt: [test.py](https://github.com/NagaraTech/gateway/blob/master/test.py)

## Pegion

Pegion is a p2p chat chrome extension base on Zeb network. The message sent by Pigeon carries VLC information during its transmission through the Zed network. By utilizing the VLC information, the propagation path of the message can be effectively determined.

## Dolphin

Dolphin is a browser for the zeb network to display a graph of message graph through nodes.

## Usage

### Integrating into the Existing Environment

1. After downloading the Chrome extension
2. add an RPC
3. create or add an account
4. then add friends to start chatting.

### Setting Up a New EnvironmentUsage

> To create a new environment, it is recommended to start with at least 10 nodes.

Each node should include a `znet` and a `zchronod` with postgresql.

[Here](https://github.com/NagaraTech/p2p-docker/tree/main/scripts) is a reference script to set up a new environment with at least 10 nodes, each including a `znet` and a `zchronod`

Sure, here's a detailed guide on how to proceed with deploying the gateway, configuring the environment, and setting up the browser and Chrome extension for chatting.

Sure, here's a detailed guide on how to proceed with deploying the gateway, configuring the environment, and setting up the browser and Chrome extension for chatting.

Create an environment configuration file (e.g., `.env`) with the necessary settings.

Here is a script to deploy the gateway using the configuration from the `.env` file:

Set the Gateway URL in the browser configuration. Here is an example of how you might configure and deploy the browser application

Ensure that your browser application reads from this configuration file and uses the `gatewayUrl` when making requests.

Deploy the browser application following your standard deployment process. This might involve building and deploying a web application.

## Benchmark

