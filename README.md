# TCP-Over-UDP

TCP-Over-UDP is a proxy library written in Go. It transfers udp packets over tcp connections. 

```
app <--udp--> client <--tcp--> server <--udp--> target
```


### Installation

```shell
go get -u github.com/justlovediaodiao/tcp-over-udp
```

Import in your code:

```go
import "github.com/justlovediaodiao/tcp-over-udp"
```

### Quick start

- client

```go
import "github.com/justlovediaodiao/tcp-over-udp"

func main() {
    // example code, ignore err check.
    conn, err := net.ListenPacket("udp", ":8080")
    client := uot.Client{
		Dialer: func(addr string) (uot.Conn, error) {
			conn, err := net.Dial("tcp", addr)
			if err != nil {
				return nil, err
			}
			return uot.DefaultOutConn(conn), nil
		},
    }
    client.Serve(uot.DefaultPacketConn(conn), "127.0.0.1:8088")
}
```

Above code listen udp packet on `:8080` and forward to `127.0.0.1:8088` over tcp.

- server

```go
import "github.com/justlovediaodiao/tcp-over-udp"

func main() {
    l, err := net.Listen("tcp", ":8088")
    var server uot.Server
    for {
		conn, err := l.Accept()
		go server.Serve(uot.DefaultInConn(conn))
	}
}
```

Above code listen tcp on `:8088` and forward to target address over udp.


### Usage

Implement `PacketConn` and `Conn` interface to define protocol.

`PacketConn` is udp packet connection. It's used by client, read udp packet, resolve target address and raw packet data.

`Conn` is tcp connection.
- In client side, send target address to server, then pack packets and forward to server. Read and unpack response packets from server.
- In server side, read target address from client, then unpack packets and forward to target address. Pack and write response packets to client.


### Default Protocol

The library provides a default implement.

Protocol define of defaultPacketConn:
```
[addr][payload]
addr: target address of packet,  which is a socks5 address defined in RFC 1928.
payload: raw udp packet.
```

Protocol define of defaultConn:
```
[handshake][packet...]
handshake: target address of packet, which is a socks5 address defined in RFC 1928.
packet: [size][payload]
size: 2-byte, length of payload.
payload: raw udp packet.
```
