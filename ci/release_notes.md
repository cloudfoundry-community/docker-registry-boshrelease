## Improvements

- The `smoke-tests` job now honors Basic Authentication (if you
  have that sort of thing set up via `proxy`) so you can test
  again!

- Better integration between the registry HTTP API and monit
  (thanks to jriguera!); Now, monit will restart the registry if
  it starts throwing HTTP 503s

- Support Google Cloud Storage as a storage backend now

- Support for docker-registry notifications add-on, to notify when
  new images are pushed / pulled.  Thanks again, jriguera.
