+++
title = "Known Issues"
date = 2023-02-21T09:00:00+02:00
description = "Known Issues in Kubermatic KubeOne"
weight = 99
+++

This page documents the list of known issues in Kubermatic KubeOne along with
possible workarounds and recommendations.

This list applies to KubeOne 1.10 release. For KubeOne 1.9, please consider
the [v1.9 version of this document][known-issues-1.9]. For earlier releases,
please consult the [appropriate changelog][changelogs].

[known-issues-1.9]: {{< ref "../../v1.9/known-issues" >}}
[changelogs]: https://github.com/kubermatic/kubeone/tree/main/CHANGELOG

## Internal Kubernetes endpoints unreachable on vSphere with Cilium/Canal

|              |                                                   |
|--------------|---------------------------------------------------|
| Status       | Workaround available                              |
| Severity     | Low                                               |
| GitHub issue | https://github.com/cilium/cilium/issues/21801     |

### Description
#### Symptoms

* Unable to perform CRUD operations on resources governed by webhooks (e.g. ValidatingWebhookConfiguration, MutatingWebhookConfiguration, etc.). The following error is observed:

```sh
Internal error occurred: failed calling webhook "webhook-name": failed to call webhook: Post "https://webhook-service-name.namespace.svc:443/webhook-endpoint": context deadline exceeded
```

* Unable to reach internal Kubernetes endpoints from pods/nodes.
* ICMP is working but TCP/UDP is not.

#### Cause

On recent enough VMware hardware compatibility version (i.e >=15 or maybe >=14), CNI connectivity breaks because of hardware segmentation offload. `cilium-health status` has ICMP connectivity working, but not TCP connectivity. cilium-health status may also fail completely.

### Recommendation

```sh
sudo ethtool -K ens192 tx-udp_tnl-segmentation off
sudo ethtool -K ens192 tx-udp_tnl-csum-segmentation off
```

These flags are related to the hardware segmentation offload done by the vSphere driver VMXNET3. We have observed this issue for both Cilium and Canal CNI running on Ubuntu 22.04.

We have two options to configure these flags for KubeOne installations:

* When configuring the VM template, set these flags as well.
* Create a [custom Operating System Profile]({{< ref "../architecture/operating-system-manager/usage#using-custom-operatingsystemprofile" >}}) and configure the flags there.

### References

* <https://github.com/cilium/cilium/issues/13096>
* <https://github.com/cilium/cilium/issues/21801>
