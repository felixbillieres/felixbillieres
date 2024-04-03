# 🧦 Sockets

## Python Sockets: A Gateway to Network Communication 🌐🔗

### Introduction to Sockets:

Sockets provide a powerful mechanism for network communication in Python. They enable data exchange between processes running on different devices over a network. Think of sockets as the endpoints for sending or receiving data.

### Creating a Simple Socket:

Let's dive into creating a basic socket. Here, we'll focus on a simple example of a server and a client.

#### Server Side:

```python
import socket

# Create a socket object
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Bind the socket to a specific address and port
server_socket.bind(("127.0.0.1", 12345))

# Listen for incoming connections
server_socket.listen()

# Accept a connection
client_socket, client_address = server_socket.accept()

# Receive data from the client
data = client_socket.recv(1024)
print("Received:", data.decode())

# Close the connection
client_socket.close()
```

#### Client Side:

```python
import socket

# Create a socket object
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to the server
client_socket.connect(("127.0.0.1", 12345))

# Send data to the server
client_socket.sendall(b"Hello, Server!")

# Close the connection
client_socket.close()
```

### Key Socket Concepts:

#### Address Family (`AF_INET`):

Specifies the address (hostname or IP address) format. `AF_INET` indicates IPv4.

#### Socket Type (`SOCK_STREAM`):

Specifies the socket type. `SOCK_STREAM` provides a reliable, stream-oriented connection.

### Practical Usage: Chat Application

Let's explore a simple chat application where a server and multiple clients can exchange messages.

#### Server Side:

```python
import socket
import threading

def handle_client(client_socket, address):
    while True:
        data = client_socket.recv(1024)
        if not data:
            break
        print(f"Received from {address}: {data.decode()}")
        # Broadcast the message to all clients
        for client in clients:
            if client != client_socket:
                client.sendall(data)

# ... (server setup code)

while True:
    client_socket, client_address = server_socket.accept()
    clients.append(client_socket)
    client_handler = threading.Thread(target=handle_client, args=(client_socket, client_address))
    client_handler.start()
```

#### Client Side:

```python
import socket
import threading

def receive_messages():
    while True:
        data = client_socket.recv(1024)
        print(f"Received: {data.decode()}")

# ... (client setup code)

message_receiver = threading.Thread(target=receive_messages)
message_receiver.start()

while True:
    message = input("Enter your message: ")
    client_socket.sendall(message.encode())
```
