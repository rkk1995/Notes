# Connection Types

## Push vs Pull

Push has a consistent connection and server pushes messages to client. Pull is wasteful and not great latency wise. Using pull, you have an inherent tradeoff between UI freshness and server load. If you increase UI freshness, server will suffer.

Try to use efficient data transfer type usch as Thrift instead of JSON. 

- JSON repeats every field name for every record. Very inefficient for large amounts of data
- JSON more geared towards human readability at cost of making it more CPU intesnive.
- Thrift has better serialization performance. 
- Thrift supports versioning.

## Web Sockets

Provides full-duplex communication channels over a single **TCP** connection. Located at layer 7 and depends on TCP at layer 4.

Pros:

- WebSockets offer bi-directional communication in realtime.
- Headers only sent once. Reduces data load sent to server.
- Binary and UTF-8.

Cons:

- When connections are terminated WebSockets don’t automatically recover. Can force client to close.
- Less overhead as connection only has to be established once.
- Enables two way data flow.

### MQTT over Websockets

PubSub system over WebSockets. The broker is the MQTT server, and the connected devices are the clients. Neither the publisher nor the clients handle the legwork. Instead, the processing power and communications are mainly handled by the broker.

- MQTT protocol is incredibly lightweight and designed to connect even the most resource-constrained of devices.
  - Low power, low bandwidth
- QualityOfService built in. Atleast once delivery.
- MQTT is bi-directional.

## Server Sent Events

Like Websockets but only one-way communication from server to client. Connection stays active

Pros:

- Transported over simple HTTP instead of a custom protocol
- Useful for apps that enable one-way communication of data, eg live stock prices

Cons:

- SSE is subject to limitation with regards to the maximum number of open connections per browser.
- UTF-8 only, no binary data.

## Long Polling

Client makes request to server and server holds connection open until new data is available to give to client. Usually closes after a tiemout. Once client receives a response, client sends another request.

- Less requests made from client

## Short Polling

Client makes requests to server repeatedly at regualr interval to check for new data. Should rarely use this.

- Making repeated requests to server wastes resources
  - each new incoming connection must be established
  - HTTP headers must be passed
  - a query for new data must be performed
  - and a response (usually with no new data to offer) must be generated and delivered. 
  - The connection must be closed and any resources cleaned up.