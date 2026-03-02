# Kubernetes (k3s) Cluster Playbook

This Ansible playbook bootstraps a Kubernetes cluster using k3s.

## Prerequisites & Execution Order

**IMPORTANT:** This playbook is designed to run *after* the `ubuntu-server-baseline` playbook has been applied to the target nodes. It depends on the baseline for essential OS hardening and configuration.

The intended execution order is:

1.  Run the `ubuntu-server-baseline/harden.yml` playbook to secure the operating system.
2.  Run this `k8s/cluster-bootstrap/v2/playbook.yml` playbook to install and configure the k3s cluster.

This playbook will fail if the baseline has not been applied, as it relies on configurations set by the baseline and includes specific overrides (like enabling IP forwarding) that are necessary for Kubernetes to function.
