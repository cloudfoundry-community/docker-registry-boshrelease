# Private Docker Registry deployed with BOSH

Run your own private Docker Registry in standalone mode (without requiring the public index).

For example, tagging and pushing the public `ubuntu:latest` repository to your Docker repository:

```plain
bosh -d docker-registry deploy \
    manifests/docker-registry.yml \
    -v ip=10.244.0.34
```

Now fetch the self-signed root CA, and the admin basic-auth password, and store in local files:

```plain
credhub get -n /bucc/docker-registry/docker_registry_certificate -j \
    | jq -r ".value.ca" > ca.pem
credhub get -n /bucc/docker-registry/docker_registry_password -j \
    | jq -r ".value" > password
```

We can now interact with the Registry via its API:

```plain
$ curl https://10.244.0.34/v2/_catalog -u "admin:$(cat password)" --cacert ca.pem
{"repositories":[]}
```

Now add ca.pem to system CA (please let use know if there's a way for `docker login` to consume a local self-signed CA). For example, in Keychain it may look like:

![keychain](https://p198.p4.n0.cdn.getcloudapp.com/items/p9u5lR81/docker-registry-ca-keychain.png?v=0a5b64b4ab0ebf3538ad61d35d45557f)

We can now `docker login` to our registry, tag `ubuntu:latest` as `10.244.0.34/ubuntu` and push it to our registry:

```bash
docker login -u admin -p "$(cat password)" 10.244.0.34
docker tag ubuntu 10.244.0.34/ubuntu
docker push 10.244.0.34/ubuntu
```

Our registry API confirms it now has the `ubuntu` image:

```plain
$ curl https://10.244.0.34/v2/_catalog -u "admin:$(cat password)"
{"repositories":["ubuntu"]}
```
