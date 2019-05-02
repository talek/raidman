Raidman
=======

Go Riemann client

```go
package main

import (
        "github.com/amir/raidman"
)

func main() {
        c, err := raidman.Dial("tcp", "localhost:5555")
        if err != nil {
                panic(err)
        }

        var event = &raidman.Event{
                State:   "success",
                Host:    "raidman",
                Service: "raidman-sample",
                Metric:  100,
                Ttl:     10,
        }

        // send one event
        err = c.Send(event)
        if err != nil {
                panic(err)
        }
        
        // send multiple events at once
        err = c.SendMulti([]*raidman.Event{
                &raidman.Event{
                        State:   "success",
                        Host:    "raidman",
                        Service: "raidman-sample",
                        Metric:  100,
                        Ttl:     10,
                },
                &raidman.Event{
                        State:   "failure",
                        Host:    "raidman",
                        Service: "raidman-sample",
                        Metric:  100,
                        Ttl:     10,
                },
                &raidman.Event{
                        State:   "success",
                        Host:    "raidman",
                        Service: "raidman-sample",
                        Metric:  100,
                        Ttl:     10,
                },
        })
        if err != nil {
                panic(err)
        }

        events, err := c.Query("host = \"raidman\"")
        if err != nil {
                panic(err)
        }

        if len(events) < 1 {
                panic("Submitted event not found")
        }

        c.Close()
}

```

How to Run the Tests Suite
--------------------------

First of all, generate certificates using [mkcert](https://github.com/FiloSottile/mkcert):

```
go get -u github.com/FiloSottile/mkcert
mkcert -install
mkcert -cert-file ~/riemann-server-crt.pem -key-file ~/riemann-server-key.pem $(domainname -f)
mkcert -cert-file ~/riemann-client-crt.pem -key-file ~/riemann-client-key.pem -client $(domainname -f)
```

Then, start a riemann server using the following config file:

```clojure
(logging/init {:console true})

(let [host "0.0.0.0"]
  (tcp-server {:host host
               :tls? true
               :key (str (System/getProperty "user.home") "/riemann-server-key.pem")
               :cert (str (System/getProperty "user.home") "/riemann-server-crt.pem")
               :ca-cert (str (System/getProperty "user.home") "/.local/share/mkcert/rootCA.pem")
               })
  (tcp-server {:host host })
  (udp-server {:host host })
)

(let [index (tap :index (index))]
  (streams
    (default :ttl 3
      (expired #(prn "Expired" %))
      (where (not (service #"^riemann "))
             index))))

(streams
  (where (= (:host event) "raidman") prn)
)

```

Run the tests:

```
go test -run ''
go test -bench=.
```
