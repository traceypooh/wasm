# docker containers, compiled into WASM, using JS in a browser

Uses the https://github.com/ktock/container2wasm setup.

This repo is a port and slight expansion of
- https://github.com/ktock/container2wasm/tree/main/examples/wasi-browser/htdocs


[![.github/workflows/workflow.yml](https://github.com/traceypooh/wasm/actions/workflows/workflow.yml/badge.svg)](https://github.com/traceypooh/wasm/actions/workflows/workflow.yml)


## Buidling `.wasm` files
NOTE: This doubles the standard ~53MB VM size to ~120MB (`128` is the default)
```sh
# https://github.com/ktock/container2wasm/blob/main/Dockerfile
./c2w  --build-arg VM_MEMORY_SIZE_MB=256  busybox  wasm/busybox.wasm
```

## Setup
These files can all be served *statically* with just about any webserver for a working demo.

**BUT** you will need two critical headers to be set to allow
[SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) to work.


### localhost example
This example uses `deno` but other web servers will work
```sh
deno run --allow-read --allow-net https://deno.land/std/http/file_server.ts
```

### Caddyfile setup example
```ini
https://example.com {
  file_server * browse {
    root /var/tmp/wasm-repo-git-clone
  }
  header Cross-Origin-Opener-Policy same-origin
  header Cross-Origin-Embedder-Policy require-corp
  header Access-Control-Allow-Origin *
  encode gzip zstd {
    match {
      header Content-Type application/wasm*
      header Content-Type application/json*
      header Content-Type application/javascript*
      header Content-Type text/*
    }
  }
}
```

## Networking
The easiset networking option is to use browser `fetch()` based networking ("out" of the container) setup.
You can try this with optional CGI arg `?net=browser`.
This has the limitation that anything you are requesting out in the "real world" needs to have
an open CORS policy.

For **full** networking, setup a "delegate" on a port that tls certs can be used with for a
`wss://` protocol, eg:
```sh
c2w-net --listen-ws localhost:8888
```

The example repo defaults to use this network setup, assuming `:8080` is used for the `wss://`
sidecar networking service.  (That port needs to be open through any firewall).

`caddy` web server can be used to easily manage the `tls` certs, eg:
```ini
https://example.com:8080 {
  reverse_proxy  localhost:8888  {
  }
}
```
