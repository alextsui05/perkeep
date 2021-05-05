---
date: 2021-05-05T00:25
---

# Running the hello app

There's a minimal server app setup here:

https://github.com/alextsui05/perkeep/blob/master/app/hello/main.go

It responds to a request with `"Hello %{word}"` where `%{word}` is a word defined in the config file:

## config.json

```json
{
  "word": "World"
}
```

The program reads the config file from the URL specified by the `CAMLI_APP_CONFIG_URL` environment variable.
Besides this, there are a handful of other environment variables expected, and it was possible to get `hello` to run with this configuration:

```
export CAMLI_APP_LISTEN=54321
export CAMLI_AUTH=username:password
export CAMLI_APP_CONFIG_URL=http://localhost:8000
export CAMLI_API_HOST=http://localhost:8000
```

* Although `CAMLI_AUTH` is required, it appears unused in `hello`.
* `CAMLI_API_HOST` needs to match the host used in `CAMLI_APP_CONFIG_URL`
* `CAMLI_API_HOST` should have the `http` prefix or else it defaults to `https`

## Serving `config.json`

`config.json` above needs to be served from `localhost:8000`, so here is a way to do that using [`gorilla/mux`](https://github.com/gorilla/mux) library:

### main.go

```go
package main

import (
  "net/http"
  "log"
  "io/ioutil"
  "github.com/gorilla/mux"
)

func YourHandler(w http.ResponseWriter, r *http.Request) {
  data, err := ioutil.ReadFile("config.json")
  if err != nil {
    panic(err)
  }
  w.Write(data)
}

func main() {
  r := mux.NewRouter()
  // Routes consist of a path and a handler function.
  r.HandleFunc("/", YourHandler)

  // Bind to a port and pass our router in
  log.Fatal(http.ListenAndServe(":8000", r))
}
```

With `config.json` in the same directory as `main.go`, run `go run main.go` and a server will respond to `localhost:8000` with the contents of `config.json`.

# Next steps

* Reference the code for `pk search` for hints on how to programmatically run blob search queries.
This is necessary to expanding the currently supported search tags.
Particularly, it would be nice to be able to [query untagged image blobs](https://groups.google.com/g/perkeep/c/rqHL4QmAxPg).