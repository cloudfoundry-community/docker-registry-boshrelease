## Improvements

- Default to binding loopback / 127.0.0.1 for the registry if no
  explicit binding IP is specified.  This makes the default
  deployment more secure, and ensures that if you are fronting
  with a colocated BOSH release providing something like nginx,
  that you don't have a side-door for bypassing that.
