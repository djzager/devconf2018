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
would look something like (after installing and configuring Minishift):

```
$ export MINISHIFT_ENABLE_EXPERIMENTAL=y
$ minishift start --openshift-version v3.7.1 --service-catalog
$ minishift addons install ./ansible-service-broker-addon
$ minishift addons apply ansible-service-broker
```

You will also need certain environment variables set:

```
$ eval $(minishift oc-env)
$ eval $(minishift docker-env)
```

### Troubleshooting

You may run into a rate limit with the GitHub API. A way to workaround this is
to use a [GitHub API Token](https://github.com/settings/tokens).

```
$ export MINISHIFT_GITHUB_API_TOKEN=$YOUR_GITHUB_API_TOKEN
```

## APB Tool Setup

You can find some great instructions
[here](https://github.com/ansibleplaybookbundle/ansible-playbook-bundle/blob/master/docs/apb_cli.md#installing-the-apb-tool)
for installing the `apb` tool, however these are geared more towards those
wanting to start active development. For the purposes of this workshop, feel
free to `pip install apb` to get up and running.
