# Kubernetes Cluster Bootstrap Playbook

This Ansible playbook automates the bootstrapping of a Kubernetes cluster on prepared nodes. It is designed to be flexible and can deploy either a lightweight `k3s` cluster or a standard `k8s` cluster.

## Features

*   **Flavor-Agnostic:** Deploy `k3s` or `k8s` by changing a single variable.
*   **CNI Installation:** Automatically installs and configures Cilium as the CNI.
*   **Essential Services:** Deploys mandatory Helm charts for:
    *   Sealed Secrets (for encrypting secrets into Git)
    *   Longhorn (distributed block storage)
    *   HashiCorp Vault (secrets management)
    *   External Secrets Operator (integrates Vault with Kubernetes secrets)
*   **Node Hardening:** Applies basic kernel and firewall settings for the cluster nodes.
*   **Sealed Secret Restore:** Restores a `SealedSecret` manifest for Longhorn TLS from an S3-compatible object store.

## Prerequisites

1.  **Ansible:** Ensure Ansible is installed on your control machine.
2.  **AWS CLI:** The `awscli` package must be installed on your control machine for the S3 tasks.
3.  **Prepared Nodes:** You should have a set of Ubuntu nodes provisioned and accessible via SSH from your control machine.
4.  **Security Hardening:** It is highly recommended to first apply the `ubuntu-server-baseline` playbook to your nodes.

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

### 3. Sealed Secret Restore for Longhorn

The playbook includes a role to restore a `SealedSecret` manifest for Longhorn's TLS certificate from an S3-compatible object store like Backblaze B2.

**Configuration:**
*   Edit the variables in `roles/sealed-secret-restore-cert/defaults/main.yml` to set your S3 bucket name and the path to the manifest file. Your S3 endpoint and credentials should be stored in the vault.

**Credentials (Ansible Vault):**
This role requires S3 credentials. The recommended way to provide them is using Ansible Vault.

1.  Create an encrypted vault file to store your secrets:
    ```bash
    ansible-vault create group_vars/vault.yml
    ```

2.  Add your provider details and credentials to the file:
    ```yaml
    # group_vars/vault.yml
    s3_endpoint_url: "https://s3.us-west-000.backblazeb2.com"
    s3_access_key_id: "YOUR_ACCESS_KEY"
    s3_secret_access_key: "YOUR_SECRET_KEY"
    ```

3.  When you run the playbook, Ansible will automatically load these variables. You will be prompted for your vault password:
    ```bash
    ansible-playbook -i inventory.yaml playbook.yml --ask-vault-pass
    ```

## Usage

Once your inventory and variables are configured, you can run the playbook:

```bash
ansible-playbook -i inventory.yaml playbook.yml --ask-vault-pass
```

After the playbook completes, the `kubeconfig` file for your new cluster will be fetched to your local `~/.kube/config`.

## Playbook Breakdown

The playbook is structured into several plays:

1.  **Apply Hardening Overrides:** Applies kernel and firewall settings specific to the cluster.
2.  **Install Prerequisites:** Installs necessary packages like `iptables` and `nfs-common` on all nodes.
3.  **Install Control Plane:** Installs the first master node (`k3s` or `k8s`).
4.  **Join Nodes:** Joins the remaining control plane and worker nodes to the cluster.
5.  **Install Cilium CNI:** Deploys and validates the Cilium CNI.
6.  **Fetch Kubeconfig:** Retrieves the `kubeconfig` from the master node.
7.  **Deploy Helm Charts:** Deploys Longhorn, Vault, and External Secrets Operator.
8.  **Restore Sealed Secret for Longhorn:** Restores the `SealedSecret` manifest for Longhorn TLS from your configured S3-compatible bucket.

## TODO

*   [ ] Create and commit a Vault manifest we can pull for correct installation.
*   [ ] Write a script to unseal Vault.