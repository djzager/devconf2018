# Creating MediaWiki APB
```
apb init mediawiki-apb
```

At this point you will see the following file structure:
```
mediawiki-apb/
├── apb.yml
├── Dockerfile
├── playbooks
│   ├── deprovision.yml
│   └── provision.yml
└── roles
    ├── deprovision-mediawiki-apb
    │   └── tasks
    │       └── main.yml
    └── provision-mediawiki-apb
        └── tasks
            └── main.yml
```

| file/directory                     | description                                                   |
|------------------------------------|---------------------------------------------------------------|
| `apb.yml`                          | The APB spec declaration                                      |
| `Dockerfile`                       | The APBs Dockerfile                                           |
| `playbooks/provision.yml`          | An Ansible Playbook defining the APBs `provision` action      |
| `playbooks/deprovision.yml`        | An Ansible Playbook defining the APBs `deprovision` action    |
| `roles/provision-mediawiki-apb`    | An Ansible Role defining which tasks are run on `provision`   |
| `roles/deprovision-mediawiki-apb`  | An Ansible Role defining which tasks are run on `deprovision` |

## APB Spec
`apb.yml` is the declaration of the APB spec. In here, we will list all relevant application specific information including: OpenShift Service Catalog metadata, all of the APBs `plans`, and input parameters to prompt the user when provisioning. Looking at apb.yml, we are free to edit the default plan that is created by the tooling. Go ahead and edit the file to match below.
```yaml
# apb.yml
version: 1.0
name: mediawiki-apb
description: This APB deploys Mediawiki123.
bindable: False
async: optional
metadata:
  displayName: Mediawiki (APB)
plans:
  - name: default
    description: This plan deploys a Mediawiki instance
    free: True
    metadata: {}
    parameters:
      - name: mediawiki_db_schema
        default: mediawiki
        type: string
        title: Mediawiki DB Schema
        required: True
      - name: mediawiki_site_name
        default: MediaWiki
        type: string
        title: Mediawiki Site Name
        required: True
        updatable: True
      - name: mediawiki_site_lang
        default: en
        type: string
        title: Mediawiki Site Language
        required: True
      - name: mediawiki_admin_user
        default: admin
        type: string
        title: Mediawiki Admin User
        required: True
      - name: mediawiki_admin_pass
        type: string
        title: Mediawiki Admin User Password
        required: True
```
Note the `parameters` section, our MediaWiki container image will expect these parameters as environment variables to be used and configured on startup. Now that we have defined our APBs parameters, we can reference them as environment variables to be injected into the container from the `deploymentConfig`.

## Provisioning
First, we need to edit the provision role for the APB. `apb init` makes this easy by leaving us commented blocks of code that we can work from.

We want three standard resources to be created by our APB: a `service`, a `deploymentConfig`, and a `route`. Start by opening up `roles/provision-mediawiki-apb/tasks/main.yml` and uncommenting the `deploymentConfig` resource so necessary information for attaching a `persistentVolume` to the MediaWiki deployment. Once you're finished editing, your `deploymentConfig` should look like this:

```yaml
- name: create deployment config
  openshift_v1_deployment_config:
    name: mediawiki
    namespace: '{{ namespace }}'
    labels:
      app: mediawiki
      service: mediawiki
    replicas: 1
    selector:
      app: mediawiki
      service: mediawiki
    spec_template_metadata_labels:
      app: mediawiki
      service: mediawiki
    containers:
    - env:
      - name: MEDIAWIKI_DB_SCHEMA
        value: "{{ mediawiki_db_schema }}"
      - name: MEDIAWIKI_SITE_NAME
        value: "{{ mediawiki_site_name }}"
      - name: MEDIAWIKI_SITE_LANG
        value: "{{ mediawiki_site_lang }}"
      - name: MEDIAWIKI_ADMIN_USER
        value: "{{ mediawiki_admin_user }}"
      - name: MEDIAWIKI_ADMIN_PASS
        value: "{{ mediawiki_admin_pass }}"
      - name: MEDIAWIKI_SITE_SERVER
      image: docker.io/ansibleplaybookbundle/mediawiki123:latest
      name: mediawiki
      ports:
      - container_port: 8080
        protocol: TCP
```

The only value we haven't set yet is the `MEDIAWIKI_SITE_SERVER` environment variable. To set this, we need to get the fully qualified route of where the application will exist on OpenShift. Let's uncomment the `route` and `service` resources that were generated and move the `route` before the `deploymentConfig` resource so we can store and use it. Your provision role should now look like this:

```yaml
- name: create mediawiki service
  k8s_v1_service:
    name: mediawiki
    namespace: '{{ namespace }}'
    labels:
      app: mediawiki
      service: mediawiki
    selector:
      app: mediawiki
      service: mediawiki
    ports:
      - name: web
        port: 8080
        target_port: 8080

- name: create mediawiki route
  openshift_v1_route:
    name: mediawiki
    namespace: '{{ namespace }}'
    spec_port_target_port: web
    labels:
      app: mediawiki
      service: mediawiki
    to_name: mediawiki
    state: present
  register: route

- name: create deployment config
  openshift_v1_deployment_config:
    name: mediawiki
    namespace: '{{ namespace }}'
    labels:
      app: mediawiki
      service: mediawiki
    replicas: 1
    selector:
      app: mediawiki
      service: mediawiki
    spec_template_metadata_labels:
      app: mediawiki
      service: mediawiki
    containers:
    - env:
      - name: MEDIAWIKI_DB_SCHEMA
        value: "{{ mediawiki_db_schema }}"
      - name: MEDIAWIKI_SITE_NAME
        value: "{{ mediawiki_site_name }}"
      - name: MEDIAWIKI_SITE_LANG
        value: "{{ mediawiki_site_lang }}"
      - name: MEDIAWIKI_ADMIN_USER
        value: "{{ mediawiki_admin_user }}"
      - name: MEDIAWIKI_ADMIN_PASS
        value: "{{ mediawiki_admin_pass }}"
      - name: MEDIAWIKI_SITE_SERVER
        value: '{{ route.route.spec.host }}'
      image: docker.io/ansibleplaybookbundle/mediawiki123:latest
      name: mediawiki
      ports:
      - container_port: 8080
        protocol: TCP
```

Note that we used `register` when we created the route to store the output of the `route` creation. We then use this output as the value for the `MEDIAWIKI_SITE_SERVER` environment variable as `{{ route.route.spec.host }}`.

## Deprovisioning
By default we recommend that an APB author provide a basic deprovision role. This allows a service consumer to delete the service from the OpenShift Service Catalog and the APBs corresponding resources to be properly deleted. `apb init` provides a skeleton deprovision role with commented resources just like the provision role. Uncomment the `service`, `route`, and `deploymentconfig` resources in the deprovision task like so:

```yaml
- openshift_v1_route:
    name: mediawiki
    namespace: '{{ namespace }}'
    state: absent

- k8s_v1_service:
    name: mediawiki
    namespace: '{{ namespace }}'
    state: absent

- openshift_v1_deployment_config:
    name: mediawiki
    namespace: '{{ namespace }}'
    state: absent
```
This will properly delete all created resources in the APBs namespace.

## Building and Testing
Now that our APB is ready to be tested, we can build and push the image to be deployed by the OpenShift Ansible Broker. For this tutorial, I am assuming your Broker is configured to source APBs from the internal OpenShift registry (Please [refer to this guide](https://github.com/openshift/ansible-service-broker/blob/master/docs/config.md#local-openshift-registry) to configure the OpenShift Ansible Broker with the `local_openshift` registry adapter).

To push your image onto the OpenShift registry, type:
```
apb push
```

This command will automatically tag & build your image and push it to the internal OpenShift registry. It will then bootstrap the broker and relist the service catalog. Refreshing the Service Catalog will display your new APB in the console. You can now deploy your application by clicking on it and filling out the respective parameters to deploy MediaWiki.
