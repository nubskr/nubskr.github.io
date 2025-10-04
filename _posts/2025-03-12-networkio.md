---
layout: post
title: "networkio: a lightweight networking protocol built over tcp"
---

[github](https://github.com/nubskr/networkio)

A lightweight application layer networking protocol that ensures resilient message delivery and connection handling without any hassle for the user.

![NetworkIO Architecture](https://raw.githubusercontent.com/nubskr/nubskr.github.io/refs/heads/master/_posts/networkio.png)

## Core Features

### Reliable Message Delivery

The protocol guarantees FIFO message delivery. Every message is considered read only when the receiving peer acknowledges it, ensuring reliable communication with built-in ACKs.

### Complete Networking Abstraction

NetworkIO handles all the networking complexity: sending, receiving, queueing, buffering, broken connections, reconnections, retransmitting messages, and ensuring reliable delivery with acknowledgments and serialization. It's all taken care of.

### Connection Resilience

Built on TCP with GOB serialization, the protocol allows peers to recognize each other after reconnection and retransmit undelivered messages. (If only real life worked like that.) The system handles thousands of active connections without any issues.

## Simple API Design

### Two Primary APIs

The application primarily deals with just two APIs. Need to send a message to a peer? It's all notification-based. Add a notification specifying who to send what to, and you'll be notified when they receive it. Similarly, when you receive something from a peer, you get notified, and they'll know for certain that you received the message.

### Lightweight Notification Queues

The notification queues are extremely lightweight, only delivering notifications. The application decides what to do with them. A notification simply informs the application that there's data to read in a connection. The application can choose to sit on it or read it immediately using a simple NetworkIO API. You can read all messages from a peer, or just one or two, whatever you need. The rest remain available to read later.

So simple even a primate could use it.

## Example Implementation

### Server Side

```go
func main() {
    networkio.AcceptConnRequestLoop("server-001")

    err := conn.WriteToConn("Hello");
    
    if err != nil {
        log.Fatalf("Failed to send message: %v", err)
    }
}
```

### Client Side

```go
// reads everything the peer sends
func worker() {
    for {
        connId := <-networkio.MasterMessageQueue
        conn, exists := networkio.Manager.GetConnFromConnId(connId)
        if exists {
            msg := conn.ReadFromConn().Data.(string)
            log.Print("new message received: ", msg)
        }
    }
}

func main() {
    go worker()

    conn, err := networkio.InitConnection("127.0.0.1", "8080","client-123")

    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }

    select {}
}
```