##  Updated packages

Updates docker distribution binary from 2.5.1 to 2.6.2.

##  New options

`docker.cache.disabled: true` disables the blobdescriptor cache.  This can be useful in CI/CD environments using this boshrelease as a pullthrough cache due to a race condition that can occur if an image is pulled before it finishes uploading to the upstream registry.
