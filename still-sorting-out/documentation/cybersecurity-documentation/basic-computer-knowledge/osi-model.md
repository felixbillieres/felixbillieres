# 🍕 OSI Model

## Comprehensive Guide to OSI Model for Beginners

### Introduction:

The OSI (Open Systems Interconnection) model is a conceptual framework that standardizes the functions of a communication system into seven distinct layers. It aims to facilitate seamless communication between different systems and devices by breaking down the complex process into manageable components.

#### Mnemonic to Remember OSI Layers:

**Mnemonic:** Please, Do, Not, Throw, Sausage, Pizza, Away

<figure><img src="http://www.tcpipguide.com/free/diagrams/osimnemonics.png" alt=""><figcaption></figcaption></figure>

#### Understanding the OSI Model:

Knowing the OSI model is invaluable for troubleshooting network issues. When receiving data, it traverses from Layer 1 to 7, and when transmitting, it moves from Layer 7 to 1. When resolving problems, layers become a useful diagnostic tool:

1. **Physical Layer (Layer 1):**
   * Responsible for transmitting raw data bits over a physical medium.
   * Defines electrical, mechanical, and functional characteristics of the physical interface.
   * Deals with cables, connectors, and physical network topology.
2. **Data Link Layer (Layer 2):**
   * Handles reliable data frame transmission between directly connected nodes.
   * Provides error detection and correction, flow control, and access to the physical medium.
   * Protocols: Ethernet, Wi-Fi, PPP.
3. **Network Layer (Layer 3):**
   * Enables routing of data packets across different networks.
   * Manages logical addressing and determines the best path for data delivery.
   * Protocols: IP (Internet Protocol).
4. **Transport Layer (Layer 4):**
   * Ensures reliable and orderly data delivery between end systems.
   * Segments data, manages end-to-end communication, and provides error recovery.
   * Protocols: TCP (Transmission Control Protocol), UDP (User Datagram Protocol).
5. **Session Layer (Layer 5):**
   * Establishes, manages, and terminates communication sessions between applications.
   * Provides synchronization, dialog control, session checkpointing, and recovery.
6. **Presentation Layer (Layer 6):**
   * Responsible for data representation, encryption, compression, and formatting.
   * Ensures data sent by one system is understandable by another.
   * Deals with data syntax and semantics.
7. **Application Layer (Layer 7):**
   * Closest layer to the end-user, providing services directly to user applications.
   * Includes protocols for various application-level services such as file transfer, email, web browsing.
   * Protocols: HTTP, SMTP, FTP, DNS.

<figure><img src="https://media.fs.com/images/community/upload/kindEditor/202107/29/original-seven-layers-of-osi-model-1627523878-JYjV8oybcC.png" alt=""><figcaption></figcaption></figure>

### Conclusion:

Understanding the OSI model is fundamental for anyone dealing with computer networks. It provides a structured approach to comprehending network communication processes, troubleshooting effectively, and ensuring interoperability between diverse systems. Use this guide as a reference to deepen your understanding of each OSI layer and its functionalities.
