# Bug Fixes

- Create the local `nfs_server.share_path` if it doesn't already
  exist, which may be the case if it's being overridden.

- Replace *ALL* occurrences of `$VCAP_UID` and `$VCAP_GID`
  variable interpolation sites in the generated /etc/exports.
