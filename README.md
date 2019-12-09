# Private Docker Registry deployed with BOSH

Run your own private Docker Registry in standalone mode (without requiring the public index).

## Simple deployment with internal DNS

The default deployment manifest will create an internal DNS hostname `docker-registry.bosh` for clients to use.

```plain
bosh -d docker-registry deploy manifests/docker-registry.yml
```

Now fetch the self-signed root CA, and the admin basic-auth password, and store in local files:

```plain
credhub get -n /bucc/docker-registry/docker_registry_certificate -j \
    | jq -r ".value.ca" > registry-ca.pem
credhub get -n /bucc/docker-registry/docker_registry_password -j \
    | jq -r ".value" > registry-password
```

We can test out our registry from within the registry's own instance. First, upload our secrets:

```plain
bosh scp registry-ca.pem registry-password docker-registry:/tmp/
```

Next, SSH into the instance:

```plain
bosh -d docker-registry ssh
```

We can now interact with the Registry via its API and its DNS alias `docker-registry.bosh`:

```plain
$ curl https://docker-registry.bosh/v2/_catalog -u "admin:$(cat /tmp/password)" --cacert /tmp/ca.pem
{"repositories":[]}
```

## Expose Docker Registry via Static IP

Delete the TLS certificate for the Docker Registry, so that a new one will be generated that includes both the new static IP, and the `docker-registry.bosh` hostname:

```plain
credhub delete -n /bucc/docker-registry/docker_registry_certificate
```

Select an available static IP from the Cloud Config. We'll use 10.244.0.34 below, and re-deploy the Docker Registry with the `manifests/operators/static-ip.yml` operator file:

```plain
bosh -d docker-registry deploy manifests/docker-registry.yml \
    -o manifests/operators/static-ip.yml \
    -v ip=10.244.0.34
```

Now add `registry-ca.pem` to system CA (please let use know if there's a way for `docker login` to consume a local self-signed CA). For example, in Keychain it may look like:

![keychain](https://p198.p4.n0.cdn.getcloudapp.com/items/p9u5lR81/docker-registry-ca-keychain.png?v=0a5b64b4ab0ebf3538ad61d35d45557f)

We can now `docker login` to our registry, tag `ubuntu:latest` as `10.244.0.34/ubuntu` and push it to our registry:

```bash
docker login -u admin -p "$(cat registry-password)" 10.244.0.34
docker tag ubuntu 10.244.0.34/ubuntu
docker push 10.244.0.34/ubuntu
```

Our registry API confirms it now has the `ubuntu` image:

```plain
$ curl https://10.244.0.34/v2/_catalog -u "admin:$(cat registry-password)"
{"repositories":["ubuntu"]}
```
