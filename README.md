# docker containers, compiled into WASM, using JS in a browser

Uses the https://github.com/ktock/container2wasm setup.

This repo is a port and slight expansion of
- https://github.com/ktock/container2wasm/tree/main/examples/wasi-browser/htdocs

with browser `fetch()` based networking ("out" of the container) setup.

## Setup
These files can all be served *statically* with just about any webserver for a working demo.

**BUT** you will need two critical headers to be set to allow
[SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) to work.


### localhost example
This example uses `deno` but other web servers will work
```sh
deno run --allow-read --allow-net https://deno.land/std/http/file_server.ts --cors -H 'Cross-Origin-Opener-Policy: same-origin' -H 'Cross-Origin-Embedder-Policy: require-corp'
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
