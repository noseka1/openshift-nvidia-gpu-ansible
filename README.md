# Ansible scripts for Deploying NVIDIA GPU Operator to OpenShift

Ansible playbooks in this repository use [NVIDIA GPU Operator](https://github.com/NVIDIA/gpu-operator). The installation process is documented in [OpenShift on NVIDIA GPU Accelerated Clusters](https://docs.nvidia.com/datacenter/kubernetes/openshift-on-gpu-install-guide/index.html).

Ansible playbooks were tested on OCP 4.4.6.

## Prerequisites

OpenShift cluster with GPU-enabled nodes:

* [Amazon EC2 G4 Instances](https://aws.amazon.com/ec2/instance-types/g4/)
* [Amazon EC2 P3 Instances](https://aws.amazon.com/ec2/instance-types/p3/)

## Deploying Node Feature Discovery Operator

Deploy NFD operator using the command:

```
$ ansible-playbook openshift_nfd.yml
```

## Installing RHEL Entitlements on OpenShift Nodes

This section is based on the instructions found at [How to use entitled image builds on Red Hat OpenShift Container Platform 4.x cluster ?](https://access.redhat.com/solutions/4908771) Follow the instructions in the Downloading certificates section of that article in order to obtain a xxxx_certificates.zip archive with certificates.

Insert the base-64 encoded content of the xxxx_certificates.zip/consumer_export.zip/export/entitlement_certificates/xxxxx.pem into the manifest ./rhel-entitlements/base/50-entitlement-key-pem-machineconfig.yaml and ./rhel-entitlements/base/50-entitlement-pem-machineconfig.yaml     .

> :exclamation: Applying the machineconfigs to the cluster will cause all worker nodes to restart.

Apply the machineconfigs to the cluster:

```
$ ansible-playbook openshift_rhel_entitlements.yml
```

Allow some time for the worker nodes to restart. Verify the RHEL entitlements configuration:

```
$ oc create --filename https://raw.githubusercontent.com/openshift-psap/blog-artifacts/master/how-to-use-entitled-builds-with-ubi/0004-cluster-wide-entitled-pod.yaml
```

```
$ oc logs cluster-entitled-build-pod
Updating Subscription Management repositories.
Unable to read consumer identity
Subscription Manager is operating in container mode.
Red Hat Enterprise Linux 8 for x86_64 - AppStre 3.2 MB/s |  18 MB     00:05
Red Hat Enterprise Linux 8 for x86_64 - BaseOS   34 MB/s |  18 MB     00:00
Red Hat Universal Base Image 8 (RPMs) - BaseOS  1.6 MB/s | 760 kB     00:00
Red Hat Universal Base Image 8 (RPMs) - AppStre 1.6 MB/s | 3.8 MB     00:02
Red Hat Universal Base Image 8 (RPMs) - CodeRea  16 kB/s | 9.1 kB     00:00
====================== Name Exactly Matched: kernel-devel ======================
kernel-devel-4.18.0-80.1.2.el8_0.x86_64 : Development package for building kernel modules to match the kernel
kernel-devel-4.18.0-80.el8.x86_64 : Development package for building kernel modules to match the kernel
kernel-devel-4.18.0-80.4.2.el8_0.x86_64 : Development package for building kernel modules to match the kernel
kernel-devel-4.18.0-80.7.1.el8_0.x86_64 : Development package for building kernel modules to match the kernel
kernel-devel-4.18.0-80.11.1.el8_0.x86_64 : Development package for building kernel modules to match the kernel
kernel-devel-4.18.0-147.el8.x86_64 : Development package for building kernel modules to match the kernel
...
```

Remove the test pod:

```
$ oc delete pod cluster-entitled-build-pod
```

## Deploying NVIDIA GPU Operator

Deploy NVIDIA GPU operator using the command:

```
$ ansible-playbook openshift_nvidia_gpu.yml
```

## References

* [OpenShift on NVIDIA GPU Accelerated Clusters](https://docs.nvidia.com/datacenter/kubernetes/openshift-on-gpu-install-guide/index.html)
* [NVIDIA GPU Operator](https://github.com/NVIDIA/gpu-operator)
* [Creating a GPU-enabled node with OpenShift 4.2 in Amazon EC2](https://www.openshift.com/blog/creating-a-gpu-enabled-node-with-openshift-4-2-in-amazon-ec2)
* [Simplifying deployments of accelerated AI workloads on Red Hat OpenShift with NVIDIA GPU Operator](https://www.openshift.com/blog/simplifying-deployments-of-accelerated-ai-workloads-on-red-hat-openshift-with-nvidia-gpu-operator)
* [How to install the NVIDIA GPU Operator with OpenShift](https://access.redhat.com/solutions/4908611)
* [How to use entitled image builds on Red Hat OpenShift Container Platform 4.x cluster ?](https://access.redhat.com/solutions/4908771)
