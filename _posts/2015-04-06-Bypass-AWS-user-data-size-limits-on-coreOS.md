---
layout: post
title: Bypassing the AWS 16KB user-data limit for CloudInit on CoreOS
---

The [AWS docs](https://coreos.com/docs/running-coreos/cloud-providers/ec2/#cloud-config) for CoreOS will get you set up quickly. Using CoreOS's example shows that we can pass in our cloud config via the page like they show in the docs or by using aws-cli.  

``` shell
aws ec2 run-instances --image-id ami-323b195a ... --user-data file://cloud-config.yaml
```

The cloud-config can quickly blow out to being a large file if you need to write out a bunch of files.

``` yaml
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new?size=3
    # specify the intial size of your cluster with ?size=X
    discovery: https://discovery.etcd.io/<token>
    # multi-region and multi-cloud deployments need to use $public_ipv4
    addr: $private_ipv4:4001
    peer-addr: $private_ipv4:7001
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
    - name: rotate-logs.service
      content: |
        [Unit]
        Description=Rotate api logs at 5:55 AM UTC

        [Timer]
        OnCalendar=05:55:00
        Unit=start-rotate-api-logs.service

        [X-Fleet]
        Global=true

write_files:
  - path: /etc/ssl/certs/site.com.key
    permissions: 0440
    owner: root
    content: |
      -----BEGIN RSA PRIVATE KEY-----
      bloggy blog blog
      -----END RSA PRIVATE KEY-----
  - path: /etc/ssl/certs/ca.crt
    permissions: 0440
    owner: root
    content: |
      -----BEGIN CERTIFICATE-----
      bloggy blog blog
      -----END CERTIFICATE-----
      -----BEGIN CERTIFICATE-----
      bloggy blog blog
      -----END CERTIFICATE-----
      -----BEGIN CERTIFICATE-----
      bloggy blog blog
      -----END CERTIFICATE-----
  - path: /etc/ssl/certs/site.com.crt
    permissions: 0440
    owner: root
    content: |
      -----BEGIN CERTIFICATE-----
      bloggy blog blog
      -----END CERTIFICATE-----
  - path: /etc/ssl/certs/site.com.pem
    permissions: 0440
    owner: root
    content: |
      -----BEGIN RSA PRIVATE KEY-----
      bloggy blog blog
      -----END RSA PRIVATE KEY-----
      -----BEGIN CERTIFICATE-----
      bloggy blog blog
      -----END CERTIFICATE-----


```

However, when trying to boot an EC2 instance with a complicated cloud-config you will probably run into a limitation within AWS of the [user-data being limited to 16KB](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html).  _The documents seem to be wrong as I was unable to send up user-data in any format that was larger than 16KB._ Once you hit this wall all AWS will tell you is:

```
A client error (InvalidParameterValue) occurred when calling the RunInstances operation: User data is limited to 16384 bytes
```

Now you can log into the instance and manually setup the machine or use some other system setup tool, but that is extra work that I would rather have coreos-cloudinit accomplish for me. To get around the limitation, I set up a systemd service to run that will download and run the real cloud-config via coreos-cloudinit.

``` yaml
#cloud-config

coreos:
  units:
    - name: bypass-user-data-limit.service
      command: start
      content: |
        [Unit]
        Description=Update the machine using our own cloud config as AWS user-data sucks

        [Service]
        EnvironmentFile=/etc/environment
        ExecStart=/usr/bin/coreos-cloudinit --from-url https://gist.github.com/KnownSubset/0b4569f401879b8c29df/raw/921e5b5f7882202ee5cf56e4875576f9409db6fd/default-cloud-config.yaml
        RemainAfterExit=yes
        Type=oneshot

```

You need the environment file to be loaded as it will supply the public and private ip addresses that coreos-cloudinit uses to do the substitutions for $public_ipv4 and $private_ipv4.
