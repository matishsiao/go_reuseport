# GO_REUSEPORT

[![Build Status](https://travis-ci.org/kavu/go_reuseport.png?branch=master)](https://travis-ci.org/kavu/go_reuseport)
[![GoDoc](https://godoc.org/github.com/kavu/go_reuseport?status.png)](https://godoc.org/github.com/kavu/go_reuseport)

**GO_REUSEPORT** is a little expirement to create a `net.Listener` that supports [SO_REUSEPORT](http://lwn.net/Articles/542629/) socket option.

For now, Darwin and Linux (from 3.9) systems are supported. I'll be pleased if you'll test other systems and tell me the results.
 documentation on [godoc.org](http://godoc.org/github.com/kavu/go_reuseport "go_reuseport documentation").

## Example ##
TCP Listener
```go
package main

import (
  "fmt"
  "html"
  "net/http"
  "os"
  "runtime"
  "github.com/kavu/go_reuseport"
)

func main() {
  runtime.GOMAXPROCS(runtime.NumCPU())

  listener, err := reuseport.NewReusablePortListener("tcp4", "localhost:8881")
  if err != nil {
    panic(err)
  }
  defer listener.Close()

  server := &http.Server{}
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Println(os.Getgid())
    fmt.Fprintf(w, "Hello, %q\n", html.EscapeString(r.URL.Path))
  })

  panic(server.Serve(listener))
}
```

UDP conn
```go
package main

import (
  "fmt"
  "html"
  "net"
  "os"
  "runtime"
  "github.com/matishsiao/go_reuseport"
)

func main() {
  runtime.GOMAXPROCS(runtime.NumCPU())

  sock, err := reuseport.NewReusableUDPPortConn("udp4", ":7900")
	 if err != nil {
		 fmt.Printf("ListenUDP Error:%v \n", err)
		 os.Exit(1)
	 }
	 receiver(sock)
}


func receiver(c net.PacketConn) {
	buf := make([]byte, 2048)
	for {
			n, addr, err := c.ReadFrom(buf)
		if err != nil {			
			return
		}
	}
}
```
Now you can run several instances of this tint server without `Address already in use` errors.

## Thanks

Inspired by [Artur Siekielski](https://github.com/aartur) [post](http://freeprogrammersblog.vhex.net/post/linux-39-introdued-new-way-of-writing-socket-servers/2) about `SO_REUSEPORT`.

