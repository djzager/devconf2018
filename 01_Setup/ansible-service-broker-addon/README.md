Ansible Service Broker Addon
======================

This addon installs the [Ansible Service Broker](https://github.com/openshift/ansible-service-broker)

# Prerequisites

This addon is designed for an RBAC enabled Openshift cluster of `>=3.7`. Minishift can be configured to
deploy 3.7 with the following command:

```
$ minishift config set openshift-version v3.7.1
```

Minishift/CDK must be deployed with the Service Catalog enabled

The Service Catalog can be deployed by adding the `--service-catalog` when starting Minishift. The ability to pass extra flags during startup can be enabled by setting the _MINISHIFT_ENABLE_EXPERIMENTAL_ environmental variable as follows:

```
$ export MINISHIFT_ENABLE_EXPERIMENTAL=y
```


Start Minishift with the `--service-catalog` flag, and `--iso-url centos`

```
$ minishift start --service-catalog --iso-url centos
```

If you are planning on building or pushing to the minishift registry, be sure
to configure your shell using `eval $(minishift docker-env)`. This ensures the
apb tooling can interface with your cluster's local registry. [See minishift documentation for more details](https://docs.openshift.org/latest/minishift/openshift/openshift-docker-registry.html).

# Deploy the Ansible Service Broker

1. Make sure this repository is cloned to the local machine
2. Install the addon


```
$ minishift addons install <path_to_addon>
```

## Addon Variables

To customize the deployment of the Ansible Service Broker, the following variables can be applied to the execution:

|Name|Description|Default Value|
|----|-----------|-------------|
|`BROKER_REPO_TAG`|Tag used to specify the broker's template in the upstream asb repo|`ansible-service-broker-1.1.6-1`|
|`APBTOOLS_REPO_TAG`|Tag used to specify the apb tooling permission template in the upstream apb repo|`apb-1.1.6-1`|
|`DOCKERHUB_ORG`|Organization to query for Ansible Playbook Bundles in DockerHub|`ansibleplaybookbundle`|

Variables can be specified by adding `--addon-env <key=value>` when the addon is being invoked (`minishift start` or `minishift addons apply`)

## Apply the Addon

To apply the addon to a running instance of Minishift, execute the following command:

```
$ minishift addons apply ansible-service-broker
```

To enable the addon each time Minishift starts, execute the following command:

```
$ minishift addons enable ansible-service-broker
```

## Remove the Addon

To remove all of the deployed components, execute the following command

```
$ minishift addons remove ansible-service-broker
```

## Uninstall the addon

To uninstall the addon, execute the following command

```
$ minishift addons uninstall ansible-service-broker
```
