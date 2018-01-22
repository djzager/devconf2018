# Binding to PostgreSQL
Now that we have a basic MediaWiki APB developed. We want to be able to bind it to a database. To do this lets create a basic PostgreSQL APB. I will not rehash all of the previous steps here but instead simply paste the APB spec and provision role.

## APB Spec
```yaml
version: 1.0
name: postgresql-apb
description: RHSCL PostgreSQL APB
bindable: true
async: optional
metadata:
  displayName: PostgreSQL (APB)
plans:
  - name: default
    description: A single DB server with no persistent storage
    free: true
    metadata:
      displayName: Default
    parameters:
      - name: postgresql_database
        default: admin
        type: string
        title: PostgreSQL Database Name
        required: true
      - name: postgresql_password
        type: string
        description: A random alphanumeric string if left blank
        title: PostgreSQL Password
      - name: postgresql_user
        default: admin
        title: PostgreSQL User
        type: string
        maxlength: 63
        required: true
```
Note how we set `bindable:` to `true`. This means that the Broker will know that our application can be bound to and we will need to provide the binding credentials at the end of our provision role.

## Provision Role
```yaml
- name: Create postgresql service
  k8s_v1_service:
    name: postgresql
    namespace: '{{ namespace }}'
    labels:
      app: postgresql
      service: postgresql
    selector:
      app: postgresql
      service: postgresql
    ports:
    - name: port-5432
      port: 5432
      protocol: TCP
      target_port: 5432
  register: postgres_service

- name: Create postgresql deployment config
  openshift_v1_deployment_config:
    name: postgresql
    namespace: '{{ namespace }}'
    labels:
      app: postgresql
      service: postgresql
    replicas: 1
    selector:
      app: postgresql
      service: postgresql
    spec_template_metadata_labels:
      app: postgresql
      service: postgresql
    containers:
    - env:
      - name: POSTGRESQL_PASSWORD
        value: '{{ postgresql_password }}'
      - name: POSTGRESQL_USER
        value: '{{ postgresql_user }}'
      - name: POSTGRESQL_DATABASE
        value: '{{ postgresql_database }}'
      image: registry.access.redhat.com/rhscl/posgresql-95-rhel7
      name: postgresql
      ports:
      - container_port: 5432
        protocol: TCP
      termination_message_path: /dev/termination-log
      working_dir: /
    triggers:
    - type: ConfigChange

- name: Wait for postgres to come up
  wait_for:
    port: 5432
    host: "{{ postgres_service.service.spec.cluster_ip }}"
    timeout: 300

- name: encode bind credentials
  asb_encode_binding:
    fields:
      DB_TYPE: postgres
      DB_HOST: postgresql
      DB_PORT: "5432"
      DB_USER: "{{ postgresql_user }}"
      DB_PASSWORD: "{{ postgresql_password }}"
      DB_NAME: "{{ postgresql_database }}"
```
The important things to note in the provision role is the final task `asb_encode_binding`. This is used on bindable applications to expose the secret fields to the broker. These variables will be injected as environment variables to the MediaWiki application container once bound. Also note that we have no need for a route in this instance, so we only create a service resource.

We will now push this image to the internal OpenShift registry:
```
apb push
```

You can now successfully bind PostgreSQL to MediaWiki in the web console.
