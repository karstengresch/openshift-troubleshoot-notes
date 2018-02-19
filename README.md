# openshift-troubleshoot-notes
Personal notes (problem - solution) on how to solve OpenShift problems I stumbled upon.

# Minishift/CDK

### Problem

  `$>docker -> could not read CA certificate "/etc/docker/ca.pem": open /etc/docker/ca.pem: no such file or directory`

### Solution (?) - F25

  `docker --tlscacert=/home/<USERNAME>/.minishift/certs/ca.pem --tlscert=/home/<USERNAME>/.minishift/certs/cert.pem --tlskey=/home/<USERNAME>/.minishift/certs/key.pem info`

(via https://github.com/minishift/minishift/issues/552)``


### Solution - F27
Edit */etc/sysconfig/docker*

Replace

   `DOCKER_CERT_PATH=/etc/docker`
   
with 
     
   ```
   if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi
   ```


### Discussion
As per the Minishift documentation, after enabling and applying the Docker registry route, you try to execute the commands advised:

```
$ eval $(minishift docker-env)
$ eval $(minishift oc-env)
$ docker login -u developer -p `oc whoami -t` docker-registry-default.192.168.42.225.nip.io:443
```

And you get the afore mentioned error.

### Problem

  `minishift start --cpus 6 --memory=10240 --service-catalog --my@user.name --ocp-tag v3.6.173.0.5`

gives

  `Error: unknown flag: --service-registry`

### Solution

  `export MINISHIFT_ENABLE_EXPERIMENTAL=y`

(via https://docs.openshift.org/latest/minishift/using/experimental-features.html)

### Discussion

### Problem
Minishift does not start due to driver check errors, e.g.

```
minishift start --cpus 7 --memory=26624 --skip-registration --ocp-tag v3.6.173.0.21
-- Checking if KVM driver is installed ... 
   Driver is available at /usr/local/bin/docker-machine-driver-kvm ... 
   Checking driver binary is executable ... OK
-- Checking if Libvirt is installed ... OK
-- Checking if Libvirt default network is present and active ... FAIL
   See the 'Setting Up the Driver Plug-in' topic for more information
```


### Solution

For KVM/libvirt on Linux, run the following command:

    `$ minishift config set warn-check-kvm-driver true`
For xhyve on macOS, run the following command:

    `$ minishift config set warn-check-xhyve-driver true`
For Hyper-V on Windows, run the following command:

    `$ minishift config set warn-check-hyperv-driver true`
    
(via https://access.redhat.com/documentation/en-us/red_hat_container_development_kit/3.2/html-single/getting_started_guide/#minishift-startup-check-failed)    

# (Specific) Images

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
 
 ### Discussion



 
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

### Discussion






### Problem: 3scale Redis pod not comming up, especially after restarting your OpenShift (Minishift) reboot 

Error description:

  `Bad file format reading the append only file: make a backup of your AOF file, then use ./redis-check-aof --fix <filename>`

https://github.com/antirez/redis/issues/3232 doesn't work as Pod doesn't come up.


### Workaround

Delete the entire project and wait:

(Minishift)

    
    oc login -u developer -p developer
    oc new-project apimanagement
    oc new-app --file https://raw.githubusercontent.com/3scale/3scale-amp-openshift-templates/master/amp/amp.yml --param WILDCARD_DOMAIN=$(minishift ip).nip.io
    

### Discussion



