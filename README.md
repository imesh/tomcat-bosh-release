# Tomcat BOSH Release

A BOSH release for deploying Apache Tomcat on BOSH implemented based on [cf-platform-eng/sample-boshrelease](https://github.com/cf-platform-eng/sample-boshrelease):

* Install BOSH Lite by following the below guide:

http://mariash.github.io/learn-bosh/

* Download the blobs for openjdk and tomcat:
```
wget https://java-buildpack.cloudfoundry.org/tomcat/tomcat-8.5.5.tar.gz
wget https://java-buildpack.cloudfoundry.org/openjdk-jdk/trusty/x86_64/openjdk-1.7.0_51.tar.gz
```
* Add the blobs:
```
bosh -e vbox add-blob openjdk-1.7.0_51.tar.gz openjdk/openjdk-1.7.0_51.tar.gz
bosh -e vbox add-blob tomcat-8.5.5.tar.gz tomcat/tomcat-8.5.5.tar.gz
bosh -e vbox -n upload-blobs
```

* Create the bosh release:
```
bosh -e vbox create-release --force
```

* Upload the bosh release to bosh:
```
bosh -e vbox upload-release
```

* Download latest bosh-lite warden stemcell from bosh.io and upload it to bosh:
``` 
wget https://s3.amazonaws.com/bosh-core-stemcells/warden/bosh-stemcell-3445.7-warden-boshlite-ubuntu-trusty-go_agent.tgz
bosh -e vbox upload-stemcell bosh-stemcell-3445.7-warden-boshlite-ubuntu-trusty-go_agent.tgz
```

* Deploy the manifest
```
bosh -e vbox -d tomcat deploy tomcat-manifest.yml
```
