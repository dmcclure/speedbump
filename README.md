# [speedbump](https://github.com/dmcclure/speedbump) [![GoDoc](https://godoc.org/github.com/dmcclure/speedbump?status.svg)](http://godoc.org/github.com/dmcclure/speedbump)

A Redis-backed Rate Limiter for Go

## Cool stuff

- Backed by Redis, so it keeps track of requests across a cluster
- Extensible timing functions. Includes defaults for tracking requests per
second, minute, and hour
- Works with IPv4, IPv6, or any other unique identifier
- Example middleware included for [Gin](https://github.com/gin-gonic/gin) (See: [ginbump](https://github.com/dmcclure/speedbump/blob/master/ginbump)) and
[Negroni](https://github.com/codegangsta/negroni) (See:
[negronibump](https://github.com/dmcclure/speedbump/blob/master/negronibump))

## Usage

- Get a working Redis server
- Include it in your code

```go
package main

import (
	"fmt"
	"time"

	"github.com/dmcclure/speedbump"
	"github.com/go-redis/redis"
)

func main() {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "",
		DB:       0,
	})
	hasher := speedbump.PerSecondHasher{}

	// Here we create a limiter that will only allow 5 requests per second
	limiter := speedbump.NewLimiter(client, hasher, 5)

	for {
		// This example has a hardcoded IP, but you would replace it with the IP
		// of a client on a real case.
		success, err := limiter.Attempt("127.0.0.1")

		if err != nil {
			panic(err)
		}

		if success {
			fmt.Println("Successful!")
		} else {
			fmt.Println("Limited! :(")
		}

		time.Sleep(time.Millisecond * time.Duration(100))
	}
}
```

- Output:

```
Successful!
Successful!
Successful!
Successful!
Successful!
Successful!
Limited! :(
Limited! :(
Limited! :(
Limited! :(
Limited! :(
Successful!
Successful!
Successful!
Successful!
Successful!
Successful!
Limited! :(
Limited! :(
Limited! :(
Limited! :(
Successful!
Successful!
...
```
