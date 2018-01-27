# Automating Application Deployment with Ansible Playbook Bundles

[Google Slides Link](https://docs.google.com/presentation/d/18pCZKg6W9r4f7wUeFQ1HV6qayP_FSFFbGf7UJDkOi5g)

## Agenda

1. What is an Ansible Playbook Bundle (APB)?
1. Create a simple APB.
1. Introduce the Ansible Broker.
1. Create a bindable APB.

## What is an Ansible Playbook Bundle (APB)

This is where we talk about the Open Service Broker and the application
primitives (provision, bind, unbind, update, deprovision). Compare/contrast
APBs with helm charts and demon working with an APB.

Include at the beginning how to get the cluster up/running/and ready with
minishift to simplify the transition.

## Create a Simple APB

This is where the workshop really begins. They'll need to install the `apb`
tool on their minishift host and get started creating mediawiki. Once they have
created an APB that can deploy mediawiki they'll provision that with `apb run`.

## Introduce the Ansible Broker

The Ansible Broker makes it easy to discover/provision existing "Services"
described in APB form. It also provides an efficient mechanism for exploring
the concept of binding a front-end like mediawiki to a back-end like postgres.
We should explore the bind/unbind concepts and what they mean with respect to
the OSB, the Ansible Broker, and APBs.

## Create a Bindable APB

Create the postgresql APB, provision it, and use the OpenShift web ui to bind
mediawiki to postgresql.
