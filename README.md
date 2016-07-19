graceful [![GoDoc](https://godoc.org/github.com/tylerb/graceful?status.png)](http://godoc.org/github.com/tylerb/graceful) [![Build Status](https://travis-ci.org/tylerb/graceful.svg?branch=master)](https://travis-ci.org/tylerb/graceful) [![Coverage Status](https://coveralls.io/repos/tylerb/graceful/badge.svg)](https://coveralls.io/r/tylerb/graceful) [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/tylerb/graceful?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
========

Graceful is  a fork of  https://github.com/tylerb/graceful
## Installation

To install, simply execute:

```
go get github.com/HiFX/graceful
```


## Usage

Using Graceful is easy. Simply create your http.Handler and pass it to the `Run` function:

```go
package main

import (
  "github.com/HiFX/graceful"
  "net/http"
  "fmt"
  "time"
)

func main() {
  mux := http.NewServeMux()
  mux.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(w, "Welcome!")
  })

  graceful.Run(":8080",10*time.Second,mux)
}
```


## Behaviour

When Graceful is sent a SIGINT or SIGTERM (possibly from ^C or a kill command), it:

1. Disables keepalive connections.
2. Closes the listening socket, allowing another process to listen on that port immediately.
3. Starts a timer of `timeout` duration to give active requests a chance to finish.
4. When timeout expires, closes all active connections.
5. Closes the `stopChan`, waking up any blocking goroutines.
6. Returns from the function, allowing the server to terminate.

## Notes

If the `timeout` argument to `Run` is 0, the server never times out, allowing all active requests to complete.

If you wish to stop the server in some way other than an OS signal, you may call the `Stop()` function.
This function stops the server, gracefully, using the new timeout value you provide. The `StopChan()` function
returns a channel on which you can block while waiting for the server to stop. This channel will be closed when
the server is stopped, allowing your execution to proceed. Multiple goroutines can block on this channel at the
same time and all will be signalled when stopping is complete.

### Important things to note when setting `timeout` to 0:

If you set the `timeout` to `0`, it waits for all connections to the server to disconnect before shutting down.
This means that even though requests over a connection have finished, it is possible for the client to hold the
connection open and block the server from shutting down indefinitely.

This is especially evident when graceful is used to run HTTP/2 servers. Clients like Chrome and Firefox have been
observed to hold onto the open connection indefinitely over HTTP/2, preventing the server from shutting down. In
addition, there is also the risk of malicious clients holding and keeping the connection alive.

It is understandable that sometimes, you might want to wait for the client indefinitely because they might be
uploading large files. In these type of cases, it is recommended that you set a reasonable timeout to kill the
connection, and have the client perform resumable uploads. For example, the client can divide the file into chunks
and reupload chunks that were in transit when the connection was terminated.

## Contributing

If you would like to contribute, please:

1. Create a GitHub issue regarding the contribution. Features and bugs should be discussed beforehand.
2. Fork the repository.
3. Create a pull request with your solution. This pull request should reference and close the issues (Fix #2).

All pull requests should:

1. Pass [gometalinter -t .](https://github.com/alecthomas/gometalinter) with no warnings.
2. Be `go fmt` formatted.
