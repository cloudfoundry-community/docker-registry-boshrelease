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
bosh upload release https://docker-registry-boshrelease.s3.amazonaws.com/boshrelease-docker-registry-1.tgz
```

To deploy it you will need the source repository that contains templates:

```
git clone https://github.com/cloudfoundry-community/docker-registry-boshrelease.git
cd docker-registry-boshrelease
```

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

### Storage backends

By default, a persistent disk is used to store the pushed docker repositories. You can also configure to use AWS S3, OpenStack Swift or Glance, and Google Compute.

See the job spec for the configuration to pass through:

- [AWS S3](https://github.com/cloudfoundry-community/docker-registry-boshrelease/blob/master/jobs/docker_registry/spec#L35-L46)
- [Swift](https://github.com/cloudfoundry-community/docker-registry-boshrelease/blob/master/jobs/docker_registry/spec#L48-L59)
- [Google Compute](https://github.com/cloudfoundry-community/docker-registry-boshrelease/blob/master/jobs/docker_registry/spec#L61-L67)

### Override security groups

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
