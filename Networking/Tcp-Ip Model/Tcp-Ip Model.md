## # `Tcp-Ip Model` -

The TCP/IP model is, in many ways, very similar to the OSI model.The TCP/IP model consists of four layers: Application, Transport, Internet and Network Interface. Between them, these cover the same range of functions as the seven layers of the OSI Model.

- <mark style="background: #FFB86CA6;">Application</mark> 
- <mark style="background: #D2B3FFA6;">Transport</mark> 
- <mark style="background: #E632B3A6;">Internet</mark> 
- <mark style="background: #07E997A6;">Network Interface</mark> 

The two models match up something like this :

![[Pasted image 20231210054318.png]]

The processes of encapsulation and de-encapsulation work in exactly the same way with the TCP/IP model as they do with the OSI model. At each layer of the TCP/IP model a header is added during encapsulation, and removed during de-encapsulation.

TCP/IP takes its name from : the **T**ransmission **C**ontrol **P**rotocol that controls the flow of data between two endpoints, and the **I**nternet **P**rotocol, which controls how packets are addressed and sent.

## # `Three-Way Handshake` -

TCP is a _connection-based_ protocol. In other words, before you send any data via TCP, you must first form a stable connection between the two computers. The process of forming this connection is called the _three-way handshake_.

When you attempt to make a connection, your computer first sends a special request to the remote server indicating that it wants to initialise a connection. This request contains something called a _SYN_ (short for _synchronise_) bit, which essentially makes first contact in starting the connection process. The server will then respond with a packet containing the SYN bit, as well as another "acknowledgement" bit, called _ACK_. Finally, your computer will send a packet that contains the ACK bit by itself, confirming that the connection has been setup successfully. With the three-way handshake successfully completed, data can be reliably transmitted between the two computers. Any data that is lost or corrupted on transmission is re-sent, thus leading to a connection which appears to be lossless.

![[Pasted image 20231210054907.png]]

