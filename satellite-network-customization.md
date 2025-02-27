---

copyright:
  years: 2023, 2023
lastupdated: "2023-06-02"

keywords: openshift, route, network, satellite

subcollection: openshift

---


{{site.data.keyword.attribute-definition-list}}



# Customizing your network setup in {{site.data.keyword.satelliteshort}} locations and clusters
{: #satellite-network-customization}

[{{site.data.keyword.satelliteshort}}]{: tag-satellite} [Red Hat CoreOS]{: tag-red}

There are several features that you can use to customize your {{site.data.keyword.satelliteshort}} network setup to better isolate and segment the services and workloads running in your location. Review the following sections for more information. 

These customizations are available only for Red Hat CoreOS-enabled locations.
{: important}

Depending on the networking customizations you want to apply, you might need to specify certain options in the CLI when creating your location, when creating your cluster, or after you have set up your location and cluster. The following tags indicate when to apply the customizations.

- [During location creation]{: tag-teal}: These customizations must be applied from the CLI during location creation.
- [During cluster creation]{: tag-dark-teal}: These customizations can be applied from the CLI during cluster creation.
- [After location and cluster creation]{: tag-green}: These customizations can be applied after creating your location and clusters.


## Defining custom subnets when creating your location
{: #sat-network-custom-subnet}

[During location creation]{: tag-teal}

When you create your location in the CLI, you can define the following parameters to customize networking in your location. For more information, see the [**`ibmcloud sat location create`**](/docs/satellite?topic=satellite-satellite-cli-reference#location-create) command reference.

You can specify the `--pod-subnet` option to specify a custom subnet CIDR to provide private IP addresses for pods. This option can be used only if you also enable Red Hat CoreOS with the `--coreos-enabled` flag. The subnet must be at least `/23` or larger. The default value is `172.16.0.0/16`.

You can also specify the `--service-subnet` option to specify a custom subnet CIDR to provide private IP addresses for services. This option can be used only if you also enable Red Hat CoreOS with the `--coreos-enabled` flag. The subnet must be at least `/24` or larger. The default value is `172.20.0.0/16`.


## Defining the pod network interface when creating your location
{: #sat-network-custom-pod-network-location}

[During location creation]{: tag-teal}

When you create your location in the CLI, you can define the `--pod-network-interface` to set the pod network interface. The available methods are `can-reach` and `interface`.

- To provide a direct URL or IP address, specify `can-reach=<url>` or `can-reach=<ip_address>`. If the network interface can reach the provided URL or IP address, this option is used. For example, use `can-reach=www.exampleurl.com` for specifying a URL and `can-reach=172.19.0.0` for specifying an IP address.
- To choose an interface with a Regex string, specify `interface=<regex_string>`; for example, `interface=eth.*`.


For more information, see the [**`ibmcloud sat location create`**](/docs/satellite?topic=satellite-satellite-cli-reference#location-create) command reference.



## Defining the pod network interface when creating your cluster
{: #sat-network-custom-pod-network-cluster}

[During cluster creation]{: tag-dark-teal}


When you create your cluster in the CLI, you can define the `--pod-network-interface` to set the pod network interface. The available methods are `can-reach` and `interface`.

- To provide a direct URL or IP address, specify `can-reach=<url>` or `can-reach=<ip_address>`. If the network interface can reach the provided URL or IP address, this option is used. For example, use `can-reach=www.exampleurl.com` for specifying a URL and `can-reach=172.19.0.0` for specifying an IP address.
- To choose an interface with a Regex string, specify `interface=<regex_string>`; for example, `interface=eth.*`.


For more information, see the [**`ibmcloud oc cluster create satellite`**](/docs/openshift?topic=openshift-kubernetes-service-cli#cli_cluster-create-satellite) command reference.

## Limiting access to your {{site.data.keyword.satelliteshort}} cluster
{: #sat-network-custom-pod-network-limit}

[After location and cluster creation]{: tag-green}


After you create your location and cluster, you can use the [**`ibmcloud ks cluster master satellite-service-endpoint allowlist add`**](/docs/openshift?topic=openshift-kubernetes-service-cli#cluster-master-sat-allowlist-add) command to add a subnet to a {{site.data.keyword.satelliteshort}} cluster's service endpoint allowlist. Authorized requests to the cluster master that originate from the subnet are permitted through the {{site.data.keyword.satelliteshort}} service endpoint. The allowlist must be [enabled](/docs/openshift?topic=openshift-kubernetes-service-cli#cluster-master-sat-allowlist-enable) for the restrictions to apply.


## Creating network policies by using Calico host endpoints
{: #custom-network-policies-hostendpoints}

[After location and cluster creation]{: tag-green}

If you create a {{site.data.keyword.satelliteshort}} cluster at version 4.12 and later, there are Calico `Hostendpoint` instances that are deployed to the cluster for every worker node’s network interface.

You can use these `Hostendpoint` instances to define global network policies with the help of the `“ibm-cloud.kubernetes.io/interface-name: <network_interface_name>”` label that is added to each `Hostendpoint` instance.

In addition to this label, all the worker node’s labels are added for additional customization options.

These `Hostendpoints` have the `“projectcalico-default-allow"` profile, which means these `Hostendpoints` may change the previously expected behavior when you update to 4.12.
{: important}

Before updating to 4.12, make sure all the previously expected networking rules, policies, `Hostendpoints` work the same after the update as well.

For more information, see the [Calico documentation](https://docs.tigera.io/calico/latest/reference/resources/hostendpoint){: external}.



## Restricting NodePort service access
{: #nodeport-restrict-access}

[After location and cluster creation]{: tag-green}

By default, NodePort services are accessible on all network interfaces that are available to the cluster, for example `0.0.0.0`.

However, in {{site.data.keyword.satelliteshort}} locations where multiple networks are available to the hosts, you can limit the available network interfaces for your services. 

To limit the range, you restrict the listening addresses for NodePort services at the cluster level. This restriction allows the cluster administrator to limit access to a specific network interface by using the IP subnet as the allowed listening address range. Complete the following steps to reconfigure the `kube-proxy` component to limit the listening address range for your NodePort services.


Incorrectly configuring the `node-port-addresses` might isolate your services from valid sources. Make sure you plan for all the required subnets your service needs. {{site.data.keyword.cloud_notm}} doesn't require access to any subnet to manage your clusters.
{: important}


1. [Access your {{site.data.keyword.redhat_openshift_notm}} cluster](/docs/openshift?topic=openshift-access_cluster).
1. Prepare your planned source subnet CIDR list that you want to allow to access the NodePort Services.
1. Run the following command to get the `network.operator.openshift.io` configuration and save a copy in case you need to revert the changes.
    ```sh
    kubectl get network.operator.openshift.io cluster -o yaml
    ```
    {: pre}
    
1. Edit the `network.operator.openshift.io` configuration and set the subnet list under the `spec` section and include the required subnets for your NodePort service.
    ```yaml
    spec:
      kubeProxyConfig:
        proxyArguments:
          node-port-addresses:
          - 192.0.2.0/24
          - 198.51.100.0/24
    ```
    {: codeblock}
    
1. Save your changes and apply them to the cluster.

    ```sh
    oc apply -f updated-network-config.yaml
    ```
    {: pre}

1. For cluster version 4.10.x and earlier, set the management state of the cluster network operator to `Unmanaged`.

   ```sh
    oc patch network.operator.openshift.io cluster --type=merge --patch  '{"spec": {"managementState": "Unmanaged"}}'
   ```

1. Restart the `kube-proxy` DaemonSet to apply the changes. This operation is not disruptive.

    ```sh
    oc rollout restart ds -n openshift-kube-proxy openshift-kube-proxy
    ```
    {: pre}
    
1. Wait until all your `kube-proxy` pods are restarted. Check the status by running the following command.
    ```sh
    oc get po -n openshift-kube-proxy --selector app=kube-proxy
    ```
    {: pre}

1. For cluster version of 4.10.x and earlier, reset the management state of the cluster network operator to `Managed`. Note that this action might restart the proxy pods.

   ```sh
    oc patch network.operator.openshift.io cluster --type=merge --patch  '{"spec": {"managementState": "Managed"}}'
   ```

After all pods are restarted, your cluster is configured with the restricted subnets. You can repeat these steps to update or remove the subnet list as needed.

You can further restrict the traffic by using `NetworkPolicies` [on a per-service basis](https://kubernetes.io/docs/concepts/services-networking/network-policies/){: external}.
{: tip}


