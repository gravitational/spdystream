# Why fork?

This repository is a fork of
[moby/spdystream](https://github.com/moby/spdystream/), focused on the
needs of [Teleport](https://github.com/gravitational/teleport).

The fork exists because:

1. It allows us to merge https://github.com/moby/spdystream/pull/91 without
	 waiting for upstream to merge it. This is a critical fix for us otherwise
	 our integration tests fail with data races.

The fork is not intended to be a long-term fork. Once the upstream repo merges
the PR and dependencies that require it are updated, we will switch to using the
upstream repo again. The fork will be deleted at that point.

The `master` branch reflects the upstream master.

The `teleport` branch is the default and reflects the library version used by
Teleport. There is no concept of semantic versioning for the `teleport` branch,
it rolls forward as Teleport needs it to.

You are looking at the `teleport` branch now.

# SpdyStream

A multiplexed stream library using spdy

## Usage

Client example (connecting to mirroring server without auth)

```go
package main

import (
	"fmt"
	"github.com/moby/spdystream"
	"net"
	"net/http"
)

func main() {
	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		panic(err)
	}
	spdyConn, err := spdystream.NewConnection(conn, false)
	if err != nil {
		panic(err)
	}
	go spdyConn.Serve(spdystream.NoOpStreamHandler)
	stream, err := spdyConn.CreateStream(http.Header{}, nil, false)
	if err != nil {
		panic(err)
	}

	stream.Wait()

	fmt.Fprint(stream, "Writing to stream")

	buf := make([]byte, 25)
	stream.Read(buf)
	fmt.Println(string(buf))

	stream.Close()
}
```

Server example (mirroring server without auth)

```go
package main

import (
	"github.com/moby/spdystream"
	"net"
)

func main() {
	listener, err := net.Listen("tcp", "localhost:8080")
	if err != nil {
		panic(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			panic(err)
		}
		spdyConn, err := spdystream.NewConnection(conn, true)
		if err != nil {
			panic(err)
		}
		go spdyConn.Serve(spdystream.MirrorStreamHandler)
	}
}
```

## Copyright and license

Copyright 2013-2021 Docker, inc. Released under the [Apache 2.0 license](LICENSE).
