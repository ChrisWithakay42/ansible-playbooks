# Kubernetes Cluster Bootstrap Playbook

This Ansible playbook automates the bootstrapping of a Kubernetes cluster on prepared nodes. It is designed to be flexible and can deploy either a lightweight `k3s` cluster or a standard `k8s` cluster.

## Features

*   **Flavor-Agnostic:** Deploy `k3s` or `k8s` by changing a single variable.
*   **Node Hardening:** Applies basic kernel and firewall settings for the cluster nodes.

## Prerequisites

1.  **Ansible:** Ensure Ansible is installed on your control machine.
2.  **Prepared Nodes:** You should have a set of Ubuntu nodes provisioned and accessible via SSH from your control machine.
3.  **Security Hardening:** It is highly recommended to first apply the `ubuntu-server-baseline` playbook to your nodes.

## Configuration

### 1. Inventory

Create an `inventory.yaml` file in this directory. You can use `../../inventory.yaml.example` as a template. Your inventory must contain the following groups:

*   `k3s_masters`: The node(s) that will form the control plane.
*   `k3s_agents`: The worker nodes.

*(Note: These group names are holdovers from the original k3s-only playbook. You can continue to use them for now, even for a k8s deployment.)*

### 2. Variables

All configuration variables are managed in the `group_vars/all.yml` file.

*   `kubernetes_flavor`: Set this to `k3s` or `k8s` to choose the desired Kubernetes distribution.
*   `kubeconfig_user`: The remote user on the master node for whom the `kubeconfig` file will be generated (e.g., `pi`).

## Usage

Once your inventory and variables are configured, you can run the playbook:

```bash
ansible-playbook -i inventory.yaml playbook.yml
```

After the playbook completes, the `kubeconfig` file for your new cluster will be fetched to your local `~/.kube/config`.

## Playbook Breakdown

The playbook is structured into several plays:

1.  **Apply Hardening Overrides:** Applies kernel and firewall settings specific to the cluster.
2.  **Install Prerequisites:** Installs necessary packages like `iptables` and `nfs-common` on all nodes.
3.  **Install Control Plane:** Installs the first master node (`k3s` or `k8s`).
4.  **Join Nodes:** Joins the remaining control plane and worker nodes to the cluster.
5.  **Fetch Kubeconfig:** Retrieves the `kubeconfig` from the master node.
