# Tomcat BOSH Release

A BOSH release for deploying Apache Tomcat on BOSH implemented based on [cf-platform-eng/sample-boshrelease](https://github.com/cf-platform-eng/sample-boshrelease):

## Quick Start Guide

* First get configuration files that specify BOSH environment in VirtualBox and run bosh create-env as following:

    ```bash
    $ git clone https://github.com/cloudfoundry/bosh-deployment bosh-deployment
    $ mkdir vbox
    $ bosh create-env bosh-deployment/bosh.yml \
    --state vbox/state.json \
    -o bosh-deployment/virtualbox/cpi.yml \
    -o bosh-deployment/virtualbox/outbound-network.yml \
    -o bosh-deployment/bosh-lite.yml \
    -o bosh-deployment/bosh-lite-runc.yml \
    -o bosh-deployment/jumpbox-user.yml \
    --vars-store vbox/creds.yml \
    -v director_name="Bosh Lite Director" \
    -v internal_ip=192.168.50.6 \
    -v internal_gw=192.168.50.1 \
    -v internal_cidr=192.168.50.0/24 \
    -v outbound_network_name=NatNetwork
    ```

* Once VM with BOSH Director is running, point your CLI to it, saving the environment with the alias vbox:

    ```bash
    bosh -e 192.168.50.6 alias-env vbox --ca-cert <(bosh int vbox/creds.yml --path /director_ssl/ca)
    ```

* Obtain generated password to BOSH Director:

    ```bash
    bosh int vbox/creds.yml --path /admin_password
    ```

* Log in using admin username and generated password:

    ```bash
    bosh -e vbox login
    ```

* Download the blobs for openjdk and tomcat:

    ```
    wget https://java-buildpack.cloudfoundry.org/tomcat/tomcat-8.5.5.tar.gz
    wget https://java-buildpack.cloudfoundry.org/openjdk-jdk/trusty/x86_64/openjdk-1.7.0_51.tar.gz
    ```

* Add the blobs:

    ```bash
    bosh -e vbox add-blob openjdk-1.7.0_51.tar.gz openjdk/openjdk-1.7.0_51.tar.gz
    bosh -e vbox add-blob tomcat-8.5.5.tar.gz tomcat/tomcat-8.5.5.tar.gz
    bosh -e vbox -n upload-blobs
    ```

* Create the Tomcat bosh release:

    ```bash
    bosh -e vbox create-release --force
    ```

* Upload the Tomcat bosh release to BOSH Director:

    ```bash
    bosh -e vbox upload-release
    ```

* Download latest bosh-lite warden stemcell from bosh.io and upload it to BOSH Director:
    
    ```bash
    wget https://s3.amazonaws.com/bosh-core-stemcells/warden/bosh-stemcell-3445.7-warden-boshlite-ubuntu-trusty-go_agent.tgz
    bosh -e vbox upload-stemcell bosh-stemcell-3445.7-warden-boshlite-ubuntu-trusty-go_agent.tgz
    ```

* Deploy the Tomcat bosh release manifest in BOSH Director:

    ```bash
    bosh -e vbox -d tomcat deploy tomcat-manifest.yml
    ```

* Add route to VirtualBox network:

    ```
    sudo route add -net 10.244.0.0/16 192.168.50.6 # Mac OS X
    sudo route add -net 10.244.0.0/16 gw 192.168.50.6 # Linux
    route add 10.244.0.0/16 192.168.50.6 # Windows
    ```
    
* Find the VM IP address via the bosh CLI and access the Tomcat web UI via a web browser:

    ```bash
    bosh -e vbox vms
    ...

    Deployment 'tomcat'
    Instance                                     Process State  AZ  IPs           VM CID                                VM Type
    tomcat/3bec9879-f727-4f5d-ad8e-3a063ba93de3  running        -   10.244.15.21  a7481310-e2f1-4060-4aef-34b1439df1e1  tomcat-resource-pool
    ...

    # Tomcat UI URL: http://10.244.15.21:8080/
    ```

## References

* [A Guide to Using BOSH](http://mariash.github.io/learn-bosh/)
* [BOSH Lite](https://bosh.io/docs/bosh-lite.html)
