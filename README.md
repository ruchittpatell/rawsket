# rawsket
raw socket connections and experiments

### resources

- c raw(7) https://man7.org/linux/man-pages/man7/raw.7.html
- wiki net sockets https://en.wikipedia.org/wiki/Network_socket
- mit sockets & networking https://web.mit.edu/6.005/www/fa16/classes/21-sockets-networking/
- tcp ip and sockets https://www.youtube.com/watch?v=C7CpfL1p6y0


### what is a socket
```
KM Ahnaf Zamil Medium Blog 
https://medium.com/@ahnafzamil/networking-101-what-are-sockets-7e2d274e153

How does it work under the hood?

When you use the socket() syscall (directly or through some library),
it ends up triggering a whole bunch of internal methods. 
These are mostly methods that run some preliminary validations
and allocates a pointer to a socket struct, which holds 
information about the socket. The socket struct contains a 
proto_ops interface that holds information about the protocol 
that the socket will use. An internal method called sock_create 
fills in necessary information about the socket in this step. 
Once these steps are done, the sock_alloc() function allocates 
an actual socket for us (it also allocates an inode but that stuff is boring).
After that, sockets are bound to IP families, ports, and whatnot (I don’t wanna go too deep into this).

Now you might be wondering, how do these sockets work?
How does the kernel do these network stuff? It’s pretty simple, actually.

Data are passed in the form of packets, so the NIC (network interface) 
doesn’t receive the data as it is. Rather, it gets them in the form of 
multiple packets with metadata in it. When some data packet is received 
by the NIC, the kernel is interrupted (or it just polls the NIC for new data).
After the kernel receives a packet from the NIC, it decodes the packet and 
looks at the metadata (in the packet headers). It checks for the source IP/port, 
destination IP/port, etc. That information is used by the kernel to look for 
an active socket in the memory that uses the same IP/port combination 
(since there can’t be more than one network socket listening on the same IP/port combo, each one is unique).

Each socket has buffers for both reading and writing, known as “receive buffer”
and “write buffer” respectively. Once the kernel figures out which socket 
the received data belongs to, it writes it to that socket’s read buffer (and holds it there).
When a user/program runs the read() syscall to read data from that socket,
the kernel copies the data from the socket’s buffer to the buffer that was supplied
when calling the read() syscall (this user-supplied buffer might be some allocated
space or variable in your code). The data that has been “read” is now 
removed from the socket’s receive buffer so that new data can come in.

Writing to a socket works similarly. When you call the write() syscall, it 
copies data from the buffer you supplied and puts it on the socket’s write 
buffer. The kernel will now take the data from the write buffer, 
serialize/encode it (check out the OSI model for more info about different network 
layers) and then hand it out the NIC which will actually send the data throughout the network.

Reading and writing can be delayed a bit from when the user calls the 
syscalls because of many reasons i.e the NIC might be busy, protocol window 
is full, kernel is busy with something else, etc.
```