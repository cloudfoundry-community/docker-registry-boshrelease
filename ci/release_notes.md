# New Features

- New "selective administration" authentication requirement.
  The new `docker.proxy.only_auth_for_admin` property lets you
  optionally open up _usage_ of the Docker Registry to anonymous
  users, but still require administrative authentication on any
  destructive (aka administrative) actions like pushing new
  images.
