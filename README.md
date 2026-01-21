# openshift-single-node-SNO
Instructions to deploy SNO
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

$ export OCP_VERSION=<ocp_version>  1

1
Replace <ocp_version> with the current version, for example, latest-4.20
Set the host architecture:

$ export ARCH=<architecture>  1

1
Replace <architecture> with the target host architecture, for example, aarch64 or x86_64.
Download the OpenShift Container Platform client (oc) and make it available for use by entering the following commands:

$ curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/openshift-client-linux.tar.gz -o oc.tar.gz


$ tar zxf oc.tar.gz

$ chmod +x oc

Download the OpenShift Container Platform installer and make it available for use by entering the following commands:

$ curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/openshift-install-linux.tar.gz -o openshift-install-linux.tar.gz


$ tar zxvf openshift-install-linux.tar.gz

$ chmod +x openshift-install

Retrieve the RHCOS ISO URL by running the following command:

$ export ISO_URL=$(./openshift-install coreos print-stream-json | grep location | grep $ARCH | grep iso | cut -d\" -f4)


Download the RHCOS ISO:

$ curl -L $ISO_URL -o rhcos-live.iso

Prepare the install-config.yaml file:

apiVersion: v1
baseDomain: <domain>  1 
compute:
- name: worker
  replicas: 0  2 
controlPlane:
  name: master
  replicas: 1  3 
metadata:
  name: <name>  4 
networking:  5 
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16  6 
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
bootstrapInPlace:
  installationDisk: /dev/disk/by-id/<disk_id>  7 
pullSecret: '<pull_secret>'  8 
sshKey: |
  <ssh_key>  9
Show less

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

$ mkdir ocp

$ cp install-config.yaml ocp

$ ./openshift-install --dir=ocp create single-node-ignition-config

Embed the ignition data into the RHCOS ISO by running the following commands:

$ alias coreos-installer='podman run --privileged --pull always --rm \
        -v /dev:/dev -v /run/udev:/run/udev -v $PWD:/data \
        -w /data quay.io/coreos/coreos-installer:release'
Show more

$ coreos-installer iso ignition embed -fi ocp/bootstrap-in-place-for-live-iso.ign rhcos-live.iso


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

$ ./openshift-install --dir=ocp wait-for install-complete

Optional: Remove the discovery ISO image.

The server restarts several times while deploying the control plane.

Verification

After the installation is complete, check the environment by running the following command:

$ export KUBECONFIG=ocp/auth/kubeconfig

$ oc get nodes

Example output


NAME                         STATUS   ROLES           AGE     VERSION
control-plane.example.com    Ready    master,worker   10m     v1.33.4

Additional resources
