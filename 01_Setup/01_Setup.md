# Setup

There are two things that will be needed to complete this workshop:

1. Minishift, to have a running OpenShift cluster
1. `apb` tool installed, to help create and run APBs

## Minishift Startup

The first things that you will need are to [install
minishift](https://docs.openshift.org/latest/minishift/getting-started/installing.html),
setup the [driver
plugin](https://docs.openshift.org/latest/minishift/getting-started/setting-up-driver-plugin.html),
and start the [ansible broker
addon](https://github.com/sabre1041/cdk-minishift-utils/tree/master/addons/ansible-service-broker).

These documents are excellent for reference, in a perfect world this setup
would look something like (after having installing and configuring Minishift):

```
$ export MINISHIFT_ENABLE_EXPERIMENTAL=y
$ minishift start --service-catalog
$ minishift addons install ./ansible-service-broker-addon
$ minishift addons apply ansible-service-broker
```

You will also need certain environment variables set:

```
$ eval $(minishift oc-env)
$ eval $(minishift docker-env)
```

## APB Tool Setup
