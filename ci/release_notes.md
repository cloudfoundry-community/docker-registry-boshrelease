# Improvements

- You can now configure the `http.relativeurls` behavior of Docker
  registry via the `docker.registry.relativeurls` manifest
  property.

- The Docker Registry configuration now honors HTTP Proxy
  configuration, via three new manifest properties:
  `docker.registry.http_proxy`, `docker.registry.https_proxy`, and
  `docker.registry.no_proxy`.
