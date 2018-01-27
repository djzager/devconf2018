# Introduction to Ansible Playbook Bundles (APBs)

APBs allow developers to leverage the power of Ansible to write, in the same
way they would write their infrastructure management, playbooks to control
their applications' lifecycle.

## Inspiration

- Solution for defining and delivering “simple” to “complex” multi-container
  applications
- Easy orchestration of services using a simple, lightweight application
  definition
- Leverage a container image as transport mechanism for delivering application
  Allowing both the application definition and container image to be hosted in
  the same location

## Background

"In a multi-cloud, multi-platform world, developers want a standard way to
connect their applications to the wealth of services available in the
marketplace." [1]

The [Open Service Broker API](https://github.com/openservicebrokerapi/servicebroker)
exists to define an open standard for meeting that need. The question then
becomes how do you package, distribute, and make these services available to
service consumers?

Enter the [Ansible Service Broker](https://github.com/openshift/ansible-service-broker),
we define services in the form of [Ansible Playbook Bundles](https://github.com/ansibleplaybookbundle/ansible-playbook-bundle)
and expose them to the [Service Catalog](https://github.com/kubernetes-incubator/service-catalog)
in OpenShift/Kubernetes.

## The Ansible Playbook Bundle Definition

1. A containerized Ansible runtime for running Ansible
1. A playbook for each of the actions that can be performed on the service
  - provision.yaml - Install
  - deprovision.yaml - Uninstall
  - bind.yaml - Grant
  - unbind.yaml - Revoke
  - update.yaml	- Upgrade
1. Metadata

# References

1. [Connect Your Applications to Azure with Open Service Broker for Azure](https://azure.microsoft.com/en-us/blog/connect-your-applications-to-azure-with-open-service-broker-for-azure/)
