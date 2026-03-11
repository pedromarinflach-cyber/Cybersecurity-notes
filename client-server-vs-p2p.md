# Client-Server vs Peer-to-Peer (P2P)

## Client-Server Model
    
The client-server model is a network architecture where clients request services and resources from a centralized server.

<img width="1536" height="1024" alt="Modelo cliente-servidor em rede" src="https://github.com/user-attachments/assets/01308913-a653-4db5-b06c-b40ef2cda2d7" />

In this model:
- The **server** provides services, resources or data to clients
- The **client** sends requests to the server and receives responses.

Examples:
- A web browser requesting a website from a web server
- Email services
- Online databases

Example workflow:
1. The client sends a request.
2. The server processes the request.
3. The server sends a response back to the client.

Advantages:
- Centralized management
- Easier security control
- Easier data backup

Disadvantages:
- If the server fails, the service may stop.
- High traffic can overload the server.


## Peer-to-Peer (P2P) Model

Peer-to-Peer is a decentralized network architecture where each device (peer) can act both as a **client and a server**.

<img width="1536" height="1024" alt="ChatGPT Image 11 de mar  de 2026, 19_37_47" src="https://github.com/user-attachments/assets/48a3035d-557b-4525-9bd6-f8365e04e5b6" />


In this model:
- Devices communicate directly with each other.
- Each peer can share resources such as files, bandwidth, or processing power.

Examples:
- File sharing networks (like BitTorrent)
- Some blockchain networks
- Distributed applications

Advantages:
- No central server required
- More resilient to single points of failure
- Can distribute network load

Disadvantages:
- Harder to control security
- Harder to manage and monitor
- Some peers may be unreliable


## Main Differences

| Feature | Client-Server | P2P |
|-------|---------------|-----|
| Structure | Centralized | Decentralized |
| Control | Central server controls services | Each peer shares responsibility |
| Reliability | Server failure can stop the service | Network continues even if peers leave |
| Management | Easier | More complex |

## Summary

- **Client-Server:** centralized architecture with a server providing services to clients.
- **P2P:** decentralized architecture where all devices can share resources and communicate directly.
