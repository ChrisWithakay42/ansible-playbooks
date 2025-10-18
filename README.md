# Homelab Playbooks

This repository contains a collection of Ansible playbooks designed to automate the setup, configuration, and management of a personal homelab environment.

## Overview

These playbooks help with:
*   Hardening Ubuntu servers to CIS benchmark standards.
*   (In progress) Bootstrapping a Kubernetes cluster (currently supporting `k3s`, `k8s` to be added in the near future).
*   (In progress) Establishing a baseline configuration for Proxmox hosts and guests.

## Directory Structure

*   `k8s/`: Contains playbooks related to Kubernetes.
    *   `cluster-bootstrap/`: Playbooks for deploying a new Kubernetes cluster.
    *   `ubuntu-server-baseline/`: A comprehensive playbook for hardening Ubuntu servers.
*   `proxmox-baseline/`: (Work in Progress) Playbooks for configuring Proxmox.

## Usage

### Prerequisites

Ensure you have Ansible installed on your control machine.

### Inventory

This repository uses an `inventory.yaml` file to define the hosts to be managed. An example file is provided at `k8s/inventory.yaml.example`.

**Important:** The `inventory.yaml` file is ignored by git to prevent accidental exposure of sensitive information.

### Playbooks

#### Ubuntu Server Baseline

This playbook applies a security hardening baseline to Ubuntu servers based on CIS recommendations.

**Location:** `k8s/ubuntu-server-baseline/`

**Roles:**
*   `common`: Disables uncommon filesystems, installs AppArmor and sudo.
*   `users`: Enforces strong password policies and account lockout.
*   `ssh`: Hardens the SSH server, disabling password authentication.
*   `firewall`: Configures `ufw` with a default-deny policy.
*   `packages`: Removes unnecessary server packages.
*   `unattended_upgrades`: Configures automatic security updates.
*   `kernel`: Hardens kernel networking parameters.
*   `auditd`: Configures the audit daemon for security logging.

**To run this playbook:**
```bash
cd k8s/ubuntu-server-baseline
ansible-playbook harden.yml
```

#### Kubernetes Cluster Bootstrap

This playbook bootstraps a Kubernetes cluster on your prepared nodes. It is designed to be flavor-agnostic, supporting both `k3s` and `k8s`.

**Location:** `k8s/cluster-bootstrap/v2/`

**Configuration:**
1.  Navigate to `k8s/cluster-bootstrap/v2/`.
2.  Create your `inventory.yaml` file.
3.  Edit `group_vars/all.yml` to set the `kubernetes_flavor` variable to either `k3s` or `k8s`.

**To run this playbook:**
```bash
cd k8s/cluster-bootstrap/v2
ansible-playbook -i inventory.yaml playbook.yml
```
