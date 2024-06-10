# ZEB Network

## Overview

Zeb network is a p2p relay network for verifying vlc clock causal order. This system is currently in poc stage, and contains several components p2p module, vlc module, gateway module, browser, chat client.

## Arch overview

![arch](./img/arch.png)

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

## API

### proto

ChatGPT
VLC and P2P use Protobuf format to exchange messages, and the same format is used between the Gateway and P2P. Here is the Protobuf format definition:

znet.proto

```protobuf
message ZMessage {
  bytes id = 1;
  uint32 version = 2;
  ZType type = 3;
  bytes public_key = 4;
  bytes data = 5;
  bytes signature = 6;
  bytes from = 7;
  bytes to = 8;
}

enum ZType {
  Z_TYPE_RNG = 0;
  Z_TYPE_EVENT = 1;
  Z_TYPE_CLOCK = 2;
  Z_TYPE_GATEWAY = 3;
  Z_TYPE_ZCHAT = 4;
}

message OutboundMsg {
  bytes id = 1;
  bytes from = 2;
  bytes to = 3;
  bytes data = 4;
  ZType type = 5;
}

message InboundMsg {
  bytes id = 1;
  bytes from = 2;
  bytes data = 3;
  ZType type = 4;
}

message NodeData {
  bytes public_key = 1;
  uint32 websocket_port = 2;
  uint32 json_rpc_port = 3;
  string domain = 4;
}
```

vlc.proto

```protobuf
syntax = "proto3";

package bussiness;

import "vlc.proto";

// business data
// ZMessage.type = Z_TYPE_CHAT
message ZChat {
	bytes message_data = 1;
	vlc.ClockInfo clock = 2;
}

// ZMessage.type = Z_TYPE_HPOINTS
message HPoints {
	bytes op_address = 1;
	bytes event_id = 2;
	uint64 points = 3;  // new points
}

// ZMessage.type = Z_TYPE_GATEWAY
// Gateway just only needs read api
message ZGateway {
    string request_id = 1;
    GatewayType type = 2;
    QueryMethod method = 3;
    bytes data = 4;
}

enum GatewayType {
  GATEWAY_TYPE_CLOCK_NODE = 0;
  GATEWAY_TYPE_MERGE_LOG = 1;   // ref merge log
  GATEWAY_TYPE_NODE_INFO = 2;  // heartbeat or node info
  GATEWAY_TYPE_Z_MESSAGE = 3;  // p2p message
}

enum QueryMethod {
    QUERY_BY_MSGID = 0;
    QUERY_BY_TABLE_KEYID = 1;
    QUERY_STATUS = 2;
}

// ZGateway.type = GATEWAY_TYPE_CLOCK_NODE
// please see ClockInfos in vlc.proto 

// ZGateway.type = GATEWAY_TYPE_NODE_INFO
message NodeInfo {
    repeated string node_ids = 1;
}

message QueryResponse {
    string request_id = 1;
    bool success = 2;
    string reason = 3;
    bytes data = 4;
}

// ZGateway.method = QUERY_BY_MSGID
message QueryByMsgID {
    string msg_id = 1;
}

// ZGateway.method = QUERY_BY_TABLE_KEYID
message QueryByTableKeyID {
    uint64 last_pos = 1;
}

// ZGateway.method = QUERY_STATUS
message QueryStatus {
    uint64 clock_total = 1;
    uint64 mergelog_total = 2;
    uint64 zmessage_total = 3;
}
```

Refer to the corresponding repositories for additional details.

### api

znet json-rpc

POST http://54.151.255.61:13333/rpc13333

```json
{
    "method": "getWsAddr",
    "address": "a308d9f92777ab8c4f5daa89592faeede951d7f5006fff600b0c082b24e964af"
}
{
    "method": "getNodeStates"
}

{
    "method": "getExtIp"
}
{
    "method": "getNeighbors"
}
{
    "id": "139812928310222",
    "method": "queryByKeyId",
    "gatewayType": 3,
    "index": 0
}
```

Definition of `gatewayType` in the `queryByKeyId` method.

```protobuf
GATEWAY_TYPE_CLOCK_NODE = 0;  // clock
GATEWAY_TYPE_MERGE_LOG = 1;   // ref merge log
GATEWAY_TYPE_NODE_INFO = 2;  // heartbeat or node info
GATEWAY_TYPE_Z_MESSAGE = 3;  // p2p message
```

gateway api is in gateway section.

## Benchmark

```shell
❯ go test -bench='^\QBenchmarkStartCluster\E$'

port: 33333, wsport: 23333, id: 406b4c9bb2117df0505a58c6c44a99c8817b7639d9c877bdbea5a8e4e0412740
port: 33334, wsport: 23334, id: f78e5a39e3d433986c4b8026d0baeb62b7eb845c29bb83a04b79d645ef7efbba
port: 33335, wsport: 23335, id: 3724b4e85737f7a77b18737535cecd676db38e88514bf0387c2d8fa62905f8eb
port: 33336, wsport: 23336, id: adcfaf62c9a25c7cb3b7fff13f0d0a3b8ef1d120fd06d25873a406a347485ce3
port: 33337, wsport: 23337, id: e8fe8612ea7f1efc861b8433caa8f8f2b193ec92459d99c4dad459d5dced0ccd
port: 33338, wsport: 23338, id: 77d04b8cef779465f06bfbc74095eea904ae3c8e307b7749850e2b71ffbbbed3
port: 33339, wsport: 23339, id: fc450038489c805d7d6dd418b1bb459ec549bf36fa89923f2250e78576440aa2
port: 33340, wsport: 23340, id: 76c0047f144a94f24115f48be88ef5ba1d45384d167ab42427f64c19a30a0dcb
port: 33341, wsport: 23341, id: ef3ea1a430356331a8e81c3d992f178b87863f9cb7195c0f1cd34418479ba9c3
port: 33342, wsport: 23342, id: 6a4c37d8ea309ea35fa29c66b36942dfd4a35da746477e39da0d997fd3646cd2
```

```shell
   20512	    106857 ns/op
--- BENCH: BenchmarkStartCluster-10
    cluster_test.go:53: 0
    cluster_test.go:40: received: 0
    cluster_test.go:53: 0
    cluster_test.go:40: received: 0
    cluster_test.go:53: 0
    cluster_test.go:40: received: 0
    cluster_test.go:53: 0
    cluster_test.go:40: received: 0
    cluster_test.go:53: 10000
    cluster_test.go:40: received: 10000
	... [output truncated]
PASS
ok  	github.com/bufrr/znet/examples	7.172s
```