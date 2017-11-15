# openshift-troubleshoot-notes
Personal notes (problem - solution) on how to solve OpenShift problems I stumbled upon.

# Minishift/CDK

### Problem

  `$>docker -> could not read CA certificate "/etc/docker/ca.pem": open /etc/docker/ca.pem: no such file or directory`

OS: F25

### Solution (I)

  `docker --tlscacert=/home/<USERNAME>/.minishift/certs/ca.pem --tlscert=/home/<USERNAME>/.minishift/certs/cert.pem --tlskey=/home/<USERNAME>/.minishift/certs/key.pem info`

(via https://github.com/minishift/minishift/issues/552)``

### Discussion




### Problem

  `minishift start --cpus 6 --memory=10240 --service-catalog --my@user.name --ocp-tag v3.6.173.0.5`

gives

  `Error: unknown flag: --service-registry`

### Solution

  `export MINISHIFT_ENABLE_EXPERIMENTAL=y`

(via https://docs.openshift.org/latest/minishift/using/experimental-features.html)

# Images

### Problem: OpenJDK image missing in default

### Solution

  ```
  oc login -u system:admin
  oc import-image my-redhat-openjdk-18/openjdk18-openshift --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift --confirm
  ```

(ImageStream definition for the Java image from https://gist.githubusercontent.com/tqvarnst/3ca512b01b7b7c1a1da0532939350e23/raw/3869a54c7dd960965f0e66907cdc3eba6d160cad/openjdk-s2i-imagestream.json )

or - if also 7.1 images are missing:
  
  ```
  oc login -u system:admin
  oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
 ```
 
### Problem: JBoss Image Stream defined, but image not resolved
 
You have defined an image stream from https://github.com/jboss-openshift/application-templates (latest and greatest).
 
Now you want to add an application to your project but, the image is not loaded. Looking up the import command from https://registry.access.redhat.com doesn't help.

When working via the UI, you get an error message like

```
An error occurred while starting the deployment. Reason: cannot trigger a deployment for "yourappname" because it contains unresolved images
```

### Solution
 
  ```
  oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
  ```
If this doesn't work, go to in the Web UI to ~Applications~->~Deployments~->Listbox ~Actions~->~Edit~ and change the image stream tag.

### Problem: 3scale Redis pod not comming up after 

Error description:

  `Bad file format reading the append only file: make a backup of your AOF file, then use ./redis-check-aof --fix <filename>`

https://github.com/antirez/redis/issues/3232 doesn't work as Pod doesn't come up.


### Workaround

Delete the entire project and wait:

(Minishift)

    
    oc login -u developer -p developer
    oc new-project apimanagement
    oc new-app --file https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/master/amp/amp.yml --param WILDCARD_DOMAIN=$(minishift ip).nip.io
    
