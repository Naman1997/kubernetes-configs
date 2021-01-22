![](https://kubernetes.io/images/kubernetes-horizontal-color.png)

![Manifest Validation](https://github.com/Naman1997/kubernetes-configs/workflows/Manifest%20Validation/badge.svg)

# Kubernetes-configs
This repo is made to learn and understand kubernetes. I'll be using a rancher instance to test all my configurations. This means all configs here are only validated for k3s clusters but not for k8s, although there should be no difference between the two.

If you want your own rancher k3s cluster please follow [this](https://rancher.com/docs/rancher/v2.x/en/installation/other-installation-methods/single-node-docker/) link. It will guide you to the simplest installation of rancher. If you need HA locally, I'd advise looking at [this](https://rancher.com/docs/rancher/v2.x/en/installation/install-rancher-on-k8s/chart-options/) link.

All folders represent some application that can be deployed within kubernetes and contains the manifests that I'll be using to deploy them locally. The folders might contain bash scripts named install.sh and uninstall.sh to quickly deploy and remove the deployment.

Each folder will also contain a Readme file that can contain useful links and post-install steps. The said Readme will also contain other details that may be required to setup the application correctly.
