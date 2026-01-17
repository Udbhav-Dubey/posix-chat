# posix-chat

This repository is a learning-focused POSIX networking project written in C++.  
It contains multiple chat implementations built while progressively learning Unix socket programming — starting from simple blocking servers to a fully event-driven, poll-based chat system.

This is **not a production chat application**.  
It is a systems programming exercise meant to understand how real networked programs handle multiple I/O sources efficiently.

---

## Repository Structure

├── blocking
│ ├── client_server_basic
│ │ ├── client.cpp
│ │ └── server.cpp
│ └── two_way_comms
│ ├── client.cpp
│ └── server.cpp
├── poll_based
│ ├── client.cpp
│ └── server.cpp
└── README.md

---

## `blocking/` — Naive Blocking Implementations

This directory contains early learning code written while first understanding POSIX sockets.

These programs use **blocking I/O** and handle one client or one operation at a time. The server blocks on calls like `accept()`, `recv()`, or `send()`, meaning it cannot respond to anything else while waiting.

### Characteristics
- Blocking `accept`, `recv`, and `send`
- One client at a time
- Sequential control flow
- Simple request/response behavior

### Purpose
This code exists intentionally to show:
- how basic TCP client/server communication works
- why blocking I/O quickly becomes limiting
- what problems arise when a server can only wait on one file descriptor

These implementations are kept as a reference point to understand *why* event-driven I/O is necessary.

---

## `poll_based/` — Event-Driven Chat System

This is the main part of the repository.

It implements a **multi-client chat room** using:
- `poll(2)` for I/O multiplexing
- non-blocking sockets (`fcntl`)
- a single-threaded event loop
- per-client read and write buffers

Both the server and the client are event-driven.

---

## Why `poll`?

Real systems rarely wait on a single thing.

A server may need to:
- accept new connections
- read from multiple clients
- write to slow clients
- detect disconnects
- remain responsive at all times

Blocking on one operation makes this impossible.

`poll` allows a program to:
- wait on multiple file descriptors simultaneously
- react only when something is ready
- avoid busy-waiting
- scale to many clients without threads

---

## How the poll-based server works

- A listening socket waits for new connections
- Each connected client socket is tracked using `pollfd`
- The event loop waits using `poll`
- Based on readiness:
  - `POLLIN` → data available to read
  - `POLLOUT` → socket ready to write buffered data
  - `POLLHUP` → client disconnected
- Messages from one client are broadcast to all others
- Each client has independent read and write buffers
- Write readiness is enabled only when there is pending data

This model closely resembles how real servers (web servers, proxies, databases, game servers) are structured internally.

---

## How the poll-based client works

The client also uses `poll` to handle two inputs at the same time:
- `STDIN` (user typing)
- the server socket

This allows:
- sending messages without blocking on server responses
- receiving messages immediately while typing
- a responsive, real-time chat experience without threads

---

## What this project is (and is not)

This project **is**:
- a systems programming learning exercise
- a clean demonstration of blocking vs event-driven I/O
- a practical introduction to I/O multiplexing

This project **is not**:
- a production-ready chat system
- performance-optimized
- using `epoll` or `kqueue` (natural next steps)

---

## Build and run (example)

```bash
# Server
g++ server.cpp -o server
./server

# Client
g++ client.cpp -o client
./client IP(use 127.0.0.1 for local) 9669
