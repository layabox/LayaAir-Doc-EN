# Network Communication

In the process of our project development, except for standalone applications that do not require network communication, when developing a network project, it is inevitable to handle network communication. This chapter will explain the network communication part of LayaAir. Usually, we use two network communication methods: HTTP and WebSocket. First, let's compare the differences between the two:

- **HTTP**:

Advantages: The protocol is mature and widely used. It is based on TCP/IP and has the reliability of TCP. The R & D cost is low, and development is fast. It is widely supported, such as by nginx, apache, tomcat, etc.

Disadvantages: It is stateless and connectionless. It only has the PULL mode and does not support PUSH. The data message is relatively large, and all header information needs to be included in each request.

Characteristics: It is stateless and connectionless (short - connection), supports the C/S mode, and is suitable for text transmission, but can also transmit other types of data, such as pictures and videos.

- **WebSocket**:

Advantages: The protocol is relatively new but has become mature. It is based on TCP/IP and has the reliability of TCP. The data message is relatively small. Only in the handshake phase when establishing a connection is relatively large header information required, and the header information is small in the subsequent data exchange phase. It is connection - oriented and a stateful protocol, and supports the server to actively push data (PUSH mode).

Disadvantages: WebSocket is an application - layer protocol. Although the data packet is more concise, compared with the TCP/IP protocol, the header information of its data packet is relatively large. For very small data packets, this may lead to some additional traffic consumption. In addition, since the connection needs to be maintained, it may consume more server resources.

Characteristics: It is stateful and connection - oriented, has a small data header, supports full - duplex communication, and is suitable for applications that require real - time communication.

Based on the above analysis of the protocol characteristics, the following suggestions are provided:

1. For weakly networked games, such as match - three games and card games, the HTTP protocol can be directly used. If security is considered, HTTPS can be used directly, or the content body can be symmetrically encrypted.
2. For applications with high requirements for real - time performance and interactivity, and if the team has relevant experience, the WebSocket protocol can be preferred, such as large - scale online games like SLG and RPG.

For specific usage introductions of these two methods, you can refer to the following two documents:

### [HTTP Communication](./HTTP/readme.md)

### [WebSocket Communication](./WebSocket/readme.md)