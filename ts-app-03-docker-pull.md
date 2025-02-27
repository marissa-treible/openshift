---

copyright:
  years: 2014, 2023
lastupdated: "2023-01-30"

keywords: openshift

subcollection: openshift
content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}




# Why can't I push or pull images from my local machine to the Docker registry?
{: #ts-app-docker-local}
{: troubleshoot}
{: support}

[Virtual Private Cloud]{: tag-vpc} [Classic infrastructure]{: tag-classic-inf}


You can't push or pull Docker images from your local machine to the cluster's built-in Docker registry.
{: tsSymptoms}


By default, the Docker registry is available internally within the cluster. You can build apps from remote directories such as GitHub or DockerHub by using the `oc new-app` command. Or you can expose your Docker registry such as with a route or load balancer so that you can push and pull images from your local machine.
{: tsCauses}


Create a route for the image registry service. For more information, see [Setting up a secure external route for the internal registry](/docs/openshift?topic=openshift-registry#route_internal_registry).
{: tsResolve}






