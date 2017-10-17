Private Docker Registry deployed with BOSH
==========================================

Run your own private Docker Registry in standalone mode (without requiring the public index) on AWS, OpenStack or vSphere.

For example, tagging and pushing the public `ubuntu:13.04` repository to an AWS-deployed Docker repository:

```
$ docker tag ubuntu:13.04 ec2-54-80-246-141.compute-1.amazonaws.com:5000/ubuntu-1304
$ docker push ec2-54-80-246-141.compute-1.amazonaws.com:5000/ubuntu-1304
The push refers to a repository [ec2-54-80-246-141.compute-1.amazonaws.com:5000/ubuntu-1304] (len: 1)
Sending image list
Pushing repository ec2-54-80-246-141.compute-1.amazonaws.com:5000/ubuntu-1304 (1 tags)
511136ea3c5a: Image successfully pushed
f323cf34fd77: Image successfully pushed
eb601b8965b8: Pushing [==================>                                ] 68.77 MB/181.2 MB 4m1s
```

Usage
-----

To use this BOSH release, first upload it to your bosh:

```
bosh upload release https://bosh.io/d/github.com/cloudfoundry-community/docker-registry-boshrelease
```

To deploy it you will need the source repository that contains templates:

```
git clone https://github.com/cloudfoundry-community/docker-registry-boshrelease.git
cd docker-registry-boshrelease
```

To use BOSH v2 manifests, have a look at `templates/docker-registry.yml`, otherwise
create the manifests using the traditional way with `templates/make_manifest`

For AWS EC2, create a single VM:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

For developers of the release using [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest and deploy:

```
templates/make_manifest warden
bosh -n deploy
```

#### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

```yaml
---
networks:
  - name: docker-registry1
    type: dynamic
    cloud_properties:
      security_groups:
        - docker_registry
```

Where `- docker_registry` means you wish to use an existing security group called `docker_registry`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```

SSL settings for Docker
-----------------------

Getting docker working with a private registry can be a time consumming
task. Those are the steps to make it easy (but also insecure):

```
# Define the IP of the Docker registry server. You could use a DNS name
# but then you have to modify the settings of the file below
REGISTRY_IP="assign the ip of the registry"

# Create a folder for the process
mkdir ssl

# Create the configuration file for openssl settings
cat <<EOF > ssl/insecure.cnf
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = NL
ST = Holland
L = Dordrecht
O = Springer Nature
CN = *

[v3_req]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:TRUE
subjectAltName = @alt_names

[alt_names]
IP = $REGISTRY_IP
DNS.1 = *
DNS.2 = *.*
DNS.3 = *.*.*
DNS.4 = *.*.*.*
DNS.5 = *.*.*.*.*
DNS.6 = *.*.*.*.*.*
DNS.7 = *.*.*.*.*.*.*

EOF

# Run openssl with the previous settings to create the self-signed certificate
openssl req -x509 -batch -nodes -newkey rsa:2048 -keyout ssl/registry.key -out ssl/registry.cert -config ssl/insecure.cnf -days 3650
```

Assign the content of the file `ssl/registry.key` to
`docker.proxy.ssl.key` property and the content of `ssl/registry.cert` to `docker.proxy.ssl.cert`. Re-deploy if it is needed.

Now it is time to configure the local docker daemon to accept the
new Registry certificates. Docker daemon uses the system certificates,
so we have to include the CA certificate there (On Ubuntu):

```
# Copy CA cert to system certificates folder
sudo cp ssl/registry.cert /usr/share/ca-certificates/
# Enable the new CA
sudo update-ca-certificates
# Restart Docker
sudo service docker restart
```

Now you can push an image to the private Docker Registry:

```
# docker push 10.230.30.76:443/bosh
The push refers to a repository [10.230.30.76:443/bosh]
5f70bf18a086: Pushed
587d5d1f5fec: Pushed
dc5fe436f9a6: Pushed
82cfb997c4aa: Pushed
a96da73480aa: Pushed
8d978c6d6037: Pushed
3fb897e02c04: Pushed
bb1d2c69066a: Pushed
5ee9df15e635: Pushed
14d1e894d916: Pushed
fe17e7c4c5c1: Pushed
3eb2e43b8463: Pushed
69e57e993b01: Pushed
e678c68d0466: Pushed
6333a97cf74a: Pushed
44599e8fdbd0: Pushed
c8bff16118ee: Pushed
62f86f3d6a38: Pushed
b4f99d457706: Pushed
f1869edf32d7: Pushed
10ee77e0916d: Pushed
0050a91818d4: Pushed
cba62530f74a: Pushed
d7a905e2ee84: Pushed
a739ea00a948: Pushed
470641744213: Pushed
latest: digest: sha256:74ca3932f4cda9b1bf31b38518e35b8cc52cda3fa6ec767dacacafc2fb27f690 size: 7208
```

Links with more information
 * https://github.com/docker/docker/issues/8849
 * https://github.com/docker/distribution/issues/1731
