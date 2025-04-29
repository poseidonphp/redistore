# redistore

[![codecov](https://codecov.io/gh/poseidonphp/redistore/branch/master/graph/badge.svg)](https://codecov.io/gh/poseidonphp/redistore)
[![Go Report Card](https://goreportcard.com/badge/github.com/poseidonphp/redistore)](https://goreportcard.com/report/github.com/poseidonphp/redistore)
[![GoDoc](https://godoc.org/github.com/poseidonphp/redistore?status.svg)](https://godoc.org/github.com/poseidonphp/redistore)
[![Run Tests](https://github.com/poseidonphp/redistore/actions/workflows/go.yml/badge.svg)](https://github.com/poseidonphp/redistore/actions/workflows/go.yml)

A session store backend for [gorilla/sessions](http://www.gorillatoolkit.org/pkg/sessions) - [src](https://github.com/gorilla/sessions).

## Requirements

Depends on the Redis [Universal Client](https://github.com/redis/go-redis) library.

You can pass a list of Redis URL's to the store, and it will automatically handle the connection pooling for you. 
It does this by parsing the URL using redis.ParseURL(). 
See the [Redis docs](https://github.com/redis/go-redis?tab=readme-ov-file) for more information.

OR if you already have a Redis client instance, you can create a new store using `NewRediStoreWithExistingClient()`. You can also create your own instance with advanced configurations, and pass that in here.

## Installation

```sh
go get github.com/poseidonphp/redistore
```

## Documentation

Available on [godoc.org](https://godoc.org/github.com/boj/redistore).

See the [repository](http://www.gorillatoolkit.org/pkg/sessions) for full documentation on underlying interface.

### Example

```go
package main

import (
  "log"
  "net/http"

  "github.com/poseidonphp/redistore"
  "github.com/gorilla/sessions"
)

func main() {
  // Fetch new store.
  store, err := redistore.NewRediStore([]string{"redis://localhost:6379"}, false, []byte("secret-key"))
  if err != nil {
    panic(err)
  }
  defer store.Close()

  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    // Get a session.
    session, err := store.Get(r, "session-key")
    if err != nil {
      log.Println(err.Error())
      return
    }

    // Add a value.
    session.Values["foo"] = "bar"

    // Save.
    if err = sessions.Save(r, w); err != nil {
      log.Fatalf("Error saving session: %v", err)
    }

    // Delete session.
    session.Options.MaxAge = -1
    if err = sessions.Save(r, w); err != nil {
      log.Fatalf("Error saving session: %v", err)
    }
  })

  log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Configuration

### SetMaxLength

Sets the maximum length of new sessions. If the length is 0, there is no limit to the size of a session.

```go
store.SetMaxLength(4096)
```

### SetKeyPrefix

Sets the prefix for session keys in Redis.

```go
store.SetKeyPrefix("myprefix_")
```

### SetSerializer

Sets the serializer for session data. The default is GobSerializer.

```go
store.SetSerializer(redistore.JSONSerializer{})
```

### SetMaxAge

Sets the maximum age, in seconds, of the session record both in the database and in the browser.

```go
store.SetMaxAge(86400 * 7) // 7 days
```

## Custom Serializers

### JSONSerializer

Serializes session data to JSON.

```go
type JSONSerializer struct{}

func (s JSONSerializer) Serialize(ss *sessions.Session) ([]byte, error) {
  // Implementation
}

func (s JSONSerializer) Deserialize(d []byte, ss *sessions.Session) error {
  // Implementation
}
```

### GobSerializer

Serializes session data using the gob package.

```go
type GobSerializer struct{}

func (s GobSerializer) Serialize(ss *sessions.Session) ([]byte, error) {
  // Implementation
}

func (s GobSerializer) Deserialize(d []byte, ss *sessions.Session) error {
  // Implementation
}
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## About This Fork

This project is a fork of [`boj/redistore`](https://github.com/boj/redistore), originally created and maintained by Bojan Marković.

The original library uses the [Redigo](https://github.com/gomodule/redigo) Redis client. This fork has been significantly modified to replace Redigo with the actively maintained [go-redis (universal client)](https://github.com/redis/go-redis).

**Key changes:**
- Replaced Redigo with `github.com/redis/go-redis/v9`
- Simplified connection handling using the UniversalClient abstraction
- Dropped support for Redigo-based connection pooling

This fork is published under the same [MIT License](./LICENSE) as the original and preserves all original licensing and attribution requirements.

If you're looking for the Redigo-based version, please refer to the [original repository](https://github.com/boj/redistore).