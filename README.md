# openshift-troubleshoot-notes
Personal notes (problem - solution) on how to solve OpenShift problems I stumbled upon.

# Minishift/CDK

### Problem

  `$>docker -> could not read CA certificate "/etc/docker/ca.pem": open /etc/docker/ca.pem: no such file or directory`

OS: F25

### Solution

  `docker --tlscacert=/home/<USERNAME>/.minishift/certs/ca.pem --tlscert=/home/<USERNAME>/.minishift/certs/cert.pem --tlskey=/home/<USERNAME>/.minishift/certs/key.pem info`

(via https://github.com/minishift/minishift/issues/552)

### Problem

  `minishift start --cpus 6 --memory=10240 --service-registry --my@user.name --ocp-tag v3.6.173.0.5`

gives

  `Error: unknown flag: --service-registry`

### Solution

  `MINISHIFT_ENABLE_EXPERIMENTAL=y`

(via https://docs.openshift.org/latest/minishift/using/experimental-features.html)
