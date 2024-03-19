# 🤝 TCP/UDP

## Understanding TCP (Transmission Control Protocol)

### Introduction to TCP:

TCP, or Transmission Control Protocol, is like the postman of the internet. It ensures that your messages (data) not only reach their destination but also get there in the right order and without any bits missing. It's all about reliability and making sure that your communication is as smooth as possible.

### TCP (Transmission Control Protocol):

#### Overview:

TCP is like a reliable postal service for your data. It ensures that your messages not only reach their destination but also get there in the right order and without any bits missing. It's all about high reliability and making sure your communication is as smooth as possible.

#### Key Characteristics:

1. **Connection-Oriented:** TCP establishes a solid connection before sending any data. This makes it ideal for applications where every bit of information is crucial and needs to be delivered accurately.
2. **3-Way Handshake:** Before data transmission begins, a polite conversation called the 3-way handshake occurs between the sender and receiver to establish a reliable connection.
3. **Example Use Cases:**
   * Websites (HTTP, HTTPS)
   * File Transfer (FTP)
   * Email (SMTP)

### UDP (User Datagram Protocol):

#### Overview:

UDP is like a fast courier service that delivers your messages quickly but doesn't stick around to ensure they've been received. It's all about speed and efficiency, making it suitable for scenarios where real-time communication matters more than ensuring every piece of data arrives.

#### Key Characteristics:

1. **Connectionless:** UDP doesn't establish a connection before sending data. It's a quick "fire and forget" approach, making it faster but less reliable than TCP.
2. **No Handshake:** Unlike TCP, UDP skips the formalities of a handshake. It sends data without waiting for a confirmation.
3. **Example Use Cases:**
   * Online Gaming (UDP's speed is crucial for real-time gaming)
   * Streaming (UDP is used for live streaming to minimize latency)
   * Voice over IP (VoIP) applications

### When to Choose TCP or UDP:

* **Choose TCP When:**
  * Reliability is crucial.
  * Data must arrive in the correct order.
  * Handshake and connection setup are acceptable overhead.
* **Choose UDP When:**
  * Speed is crucial, and some data loss is acceptable.
  * Real-time communication is a priority.
  * Overhead of connection setup is not acceptable.

### Comparison Summary:

| Feature               | TCP                            | UDP                            |
| --------------------- | ------------------------------ | ------------------------------ |
| **Reliability**       | High                           | Lower                          |
| **Connection Setup**  | 3-Way Handshake                | No Handshake                   |
| **Order of Delivery** | Yes (Ensured)                  | No (Not Ensured)               |
| **Use Cases**         | Websites, File Transfer, Email | Online Gaming, Streaming, VoIP |

### The 3-Way Handshake:

Imagine you're sending a fancy invitation to a friend. You want to make sure they receive it, so you use a reliable courier service (TCP). The 3-way handshake is like a polite conversation between your computer and the server to establish this reliable connection.

1. **SYN (Synchronize):** Your computer sends a message to the server, saying, "Hey, I want to talk, and I'm starting the conversation. Are you there?"
2. **SYN-ACK (Synchronize-Acknowledge):** The server replies, "Yes, I got your message, and I'm ready to talk. Let's synchronize our communication."
3. **ACK (Acknowledge):** Your computer acknowledges the server's response, saying, "Great! We are synchronized, and now we can start sending data back and forth."

<figure><img src="https://miro.medium.com/v2/resize:fit:1102/1*IE_SkdCYUrY3HyQ5U59WZg.png" alt=""><figcaption></figcaption></figure>

### Wireshark Example:

Ever wondered what's happening under the hood? Wireshark is like an x-ray for your internet communication. Here's a snapshot of a 3-way handshake in Wireshark:

<figure><img src="https://infraexam.com/wp-content/uploads/2020/11/5fb3fd530fd1f_img.png" alt=""><figcaption><p>SYN-SYN/ACK-ACK</p></figcaption></figure>

In this friendly conversation, your computer and the server ensure they're on the same page before exchanging important information.

TCP is your go-to protocol when you want your data to arrive intact and in the right order. It's the careful messenger that makes sure your internet experience is reliable and smooth.
