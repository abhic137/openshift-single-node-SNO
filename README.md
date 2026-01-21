# openshift-single-node-SNO
Instructions to deploy SNO version 4.20
Refrence: ```https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/installing_on_a_single_node/install-sno-installing-sno#install-sno-generating-the-discovery-iso-with-the-assisted-installer_install-sno-installing-sno-with-the-assisted-installer```
# Installing single-node OpenShift manually 

To install OpenShift Container Platform on a single node, first generate the installation ISO, and then boot the server from the ISO. You can monitor the installation using the openshift-install installation program.

Additional resources

Networking requirements for user-provisioned infrastructure
User-provisioned DNS requirements
Configuring DHCP or static IP addresses
## Generating the installation ISO with coreos-installer 

Installing OpenShift Container Platform on a single node requires an installation ISO, which you can generate with the following procedure.

Prerequisites

Install podman.
Note
See "Requirements for installing OpenShift on a single node" for networking requirements, including DNS records.

Procedure

Set the OpenShift Container Platform version:
```
$ export OCP_VERSION=<ocp_version>  1
```
1
Replace <ocp_version> with the current version, for example, latest-4.20
Set the host architecture:
```
$ export ARCH=<architecture>  1
```
1
Replace <architecture> with the target host architecture, for example, aarch64 or x86_64.
Download the OpenShift Container Platform client (oc) and make it available for use by entering the following commands:
```
$ curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/openshift-client-linux.tar.gz -o oc.tar.gz


$ tar zxf oc.tar.gz

$ chmod +x oc
```
Download the OpenShift Container Platform installer and make it available for use by entering the following commands:
```
$ curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/openshift-install-linux.tar.gz -o openshift-install-linux.tar.gz


$ tar zxvf openshift-install-linux.tar.gz

$ chmod +x openshift-install

Retrieve the RHCOS ISO URL by running the following command:

$ export ISO_URL=$(./openshift-install coreos print-stream-json | grep location | grep $ARCH | grep iso | cut -d\" -f4)

```
Download the RHCOS ISO:
```
$ curl -L $ISO_URL -o rhcos-live.iso
```
Prepare the install-config.yaml file:
```
apiVersion: v1
baseDomain: <domain>  #1 
compute:
- name: worker
  replicas: 0  #2 
controlPlane:
  name: master
  replicas: 1  #3 
metadata:
  name: <name>  #4 
networking:  5 
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16 #6 
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
bootstrapInPlace:
  installationDisk: /dev/disk/by-id/<disk_id>  #7 
pullSecret: '<pull_secret>'  #8 
sshKey: |
  <ssh_key>  #9

```
1
Add the cluster domain name.
2
Set the compute replicas to 0. This makes the control plane node schedulable.
3
Set the controlPlane replicas to 1. In conjunction with the previous compute setting, this setting ensures the cluster runs on a single node.
4
Set the metadata name to the cluster name.
5
Set the networking details. OVN-Kubernetes is the only allowed network plugin type for single-node clusters.
6
Set the cidr value to match the subnet of the single-node OpenShift cluster.
7
Set the path to the installation disk drive, for example, /dev/disk/by-id/wwn-0x64cd98f04fde100024684cf3034da5c2.
8
Copy the pull secret from Red Hat OpenShift Cluster Manager and add the contents to this configuration setting.
9
Add the public SSH key from the administration host so that you can log in to the cluster after installation.
Generate OpenShift Container Platform assets by running the following commands:
```
$ mkdir ocp

$ cp install-config.yaml ocp

$ ./openshift-install --dir=ocp create single-node-ignition-config
```
Embed the ignition data into the RHCOS ISO by running the following commands:
```
$ alias coreos-installer='podman run --privileged --pull always --rm \
        -v /dev:/dev -v /run/udev:/run/udev -v $PWD:/data \
        -w /data quay.io/coreos/coreos-installer:release'


$ coreos-installer iso ignition embed -fi ocp/bootstrap-in-place-for-live-iso.ign rhcos-live.iso

```
Important
The SSL certificates for the RHCOS ISO installation image are only valid for 24 hours. If you use the ISO image to install a node more than 24 hours after creating the image, the installation can fail. To re-create the image after 24 hours, delete the ocp directory and re-create the OpenShift Container Platform assets.

Additional resources

See Requirements for installing OpenShift on a single node for more information about installing OpenShift Container Platform on a single node.
See Cluster capabilities for more information about enabling cluster capabilities that were disabled before installation.
See Optional cluster capabilities in OpenShift Container Platform 4.20 for more information about the features provided by each capability.
2.2.2. Monitoring the cluster installation using openshift-install 

Use openshift-install to monitor the progress of the single-node cluster installation.

Prerequisites

Ensure that the boot drive order in the server BIOS settings defaults to booting the server from the target installation disk.
Procedure

Attach the discovery ISO image to the target host.
Boot the server from the discovery ISO image. The discovery ISO image writes the system configuration to the target installation disk and automatically triggers a server restart.
On the administration host, monitor the installation by running the following command:
```
$ ./openshift-install --dir=ocp wait-for install-complete
```
Optional: Remove the discovery ISO image.

The server restarts several times while deploying the control plane.

Verification

After the installation is complete, check the environment by running the following command:
```
$ export KUBECONFIG=ocp/auth/kubeconfig

$ oc get nodes
```
Example output
```

NAME                         STATUS   ROLES           AGE     VERSION
control-plane.example.com    Ready    master,worker   10m     v1.33.4
```
Additional resources

See Requirements for installing OpenShift on a single node for more information about installing OpenShift Container Platform on a single node.
```
https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/installing_on_a_single_node/preparing-to-install-sno
```
See Cluster capabilities for more information about enabling cluster capabilities that were disabled before installation.
```
https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/installation_overview/#cluster-capabilities
```
See Optional cluster capabilities in OpenShift Container Platform 4.20 for more information about the features provided by each capability.
```
https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/installation_overview/#explanation_of_capabilities_cluster-capabilities
```



[id="install-sno-installing-sno-manually"]
== Installing {sno} manually
endif::openshift-origin[]
ifdef::openshift-origin[]
[id="install-sno-installing-sno-manually"]
== Installing {sno-okd} manually
endif::openshift-origin[]

To install {product-title} on a single node, first generate the installation ISO, and then boot the server from the ISO. You can monitor the installation using the `openshift-install` installation program.

[role="_additional-resources"]
.Additional resources

* xref:../../installing/installing_bare_metal/upi/installing-bare-metal-network-customizations.adoc#installation-network-user-infra_installing-bare-metal-network-customizations[Networking requirements for user-provisioned infrastructure]

* xref:../../installing/installing_bare_metal/upi/installing-bare-metal-network-customizations.adoc#installation-dns-user-infra_installing-bare-metal-network-customizations[User-provisioned DNS requirements]

* xref:../../installing/installing_bare_metal/upi/installing-bare-metal-network-customizations.adoc#configuring-dhcp-or-static-ip-addresses_installing-bare-metal-network-customizations[Configuring DHCP or static IP addresses]

include::modules/install-sno-generating-the-install-iso-manually.adoc[leveloffset=+2]

[role="_additional-resources"]
.Additional resources

* See xref:../../installing/installing_sno/install-sno-preparing-to-install-sno.adoc#preparing-to-install-sno[Requirements for installing OpenShift on a single node] for more information about installing {product-title} on a single node.
* See xref:../../installing/overview/cluster-capabilities.adoc#cluster-capabilities[Cluster capabilities] for more information about enabling cluster capabilities that were disabled before installation.
* See xref:../../installing/overview/cluster-capabilities.adoc#explanation_of_capabilities_cluster-capabilities[Optional cluster capabilities in {product-title} {product-version}] for more information about the features provided by each capability.

// Monitoring the cluster installation using openshift-install
include::modules/install-sno-monitoring-the-installation-manually.adoc[leveloffset=+2]

[role="_additional-resources"]
.Additional resources

* xref:../../installing/installing_sno/install-sno-installing-sno.adoc#installing-with-usb-media_install-sno-installing-sno-with-the-assisted-installer[Creating a bootable ISO image on a USB drive]
ifndef::openshift-origin[]
* xref:../../installing/installing_sno/install-sno-installing-sno.adoc#install-booting-from-an-iso-over-http-redfish_install-sno-installing-sno-with-the-assisted-installer[Booting from an HTTP-hosted ISO image using the Redfish API]
endif::openshift-origin[]
ifndef::openshift-origin[]
* xref:../../nodes/nodes/nodes-sno-worker-nodes.adoc#nodes-sno-worker-nodes[Adding worker nodes to {sno} clusters]

