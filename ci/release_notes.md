# Improvements

- The `proxy` job can now be configured to specify exact TLS versions
  to enable via the `docker.proxy.ssl.protocols` property. This defaults
  to `TLSv1.1 TLSv1.2`. To enable TLS 1.2 only, change the value to `TLSv1.2`.
- The `proxy` job can now be configured with a custom DH param PEM via the
  `docker.proxy.ssl.dhparam` property. If the property is omitted, nginx's
  default dhparam settings will be used. 
