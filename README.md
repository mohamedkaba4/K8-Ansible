# Kubernetes Infrastructure Automation

Ansible-based infrastructure automation for a multi-node Kubernetes home lab running across two Mac hosts using UTM virtual machines.

## Overview

This project automates Kubernetes node provisioning, configuration, and worker onboarding. GitHub Actions provides a centralized workflow for running infrastructure operations through a self-hosted runner.

**Current environment:**
- Kubernetes v1.36.5, deployed using kubeadm
- 1 control plane and 4 worker nodes
- 2 additional control-plane VMs reserved for future HA
- Ubuntu Linux VMs (ARM64 and AMD64)
- Tailscale for cross-host connectivity
- Flannel CNI for Pod networking

## Architecture

```text
GitHub Actions
      |
Self-Hosted Runner (Mac)
      |
    Ansible
      |
   Tailscale
      |
Kubernetes Infrastructure
├── control1 (active)
├── worker1
├── worker2
├── worker3
├── worker4
├── control2 (standby, not joined)
└── control3 (standby, not joined)
```

## Technologies

| Technology | Purpose |
|---|---|
| Ansible | Infrastructure configuration and automation |
| GitHub Actions | CI/CD workflow orchestration |
| Kubernetes / kubeadm | Cluster orchestration and node management |
| Tailscale | Secure connectivity between VMs |
| Flannel | Kubernetes Pod networking |
| containerd | Container runtime |

## Repository Structure

```text
.
├── .github/workflows/
│   └── infrastructure.yml
├── inventory/
│   └── hosts.ini
├── playbooks/
│   ├── prerequisites.yml
│   ├── containerd.yml
│   ├── kubernetes.yml
│   ├── join-workers.yml
│   ├── tailscale-node-ips.yml
│   └── ansible-sudo.yml
└── provision.yml
```

## Usage

**Verify connectivity to all managed VMs:**

```bash
ansible -i inventory/hosts.ini k8s -m ping
```

**Provision a new node:**

```bash
ansible-playbook -i inventory/hosts.ini provision.yml \
  --limit worker5
```

**Join a worker to Kubernetes:**

```bash
ansible-playbook -i inventory/hosts.ini \
  playbooks/join-workers.yml \
  -e target_worker=worker5
```

New nodes must first be added to the Ansible inventory and configured for SSH and Tailscale connectivity.

## GitHub Actions

The manual infrastructure workflow supports three operations:

- **Validate:** Verify SSH connectivity, Kubernetes node readiness, and Flannel health.
- **Provision:** Install Kubernetes prerequisites and required packages on a selected node.
- **Join Worker:** Register a selected worker with the existing cluster.

The workflow executes on a self-hosted macOS runner with network access to the VMs.

## Project Highlights

- Centralized management of seven Ubuntu VMs across two physical hosts.
- Automated Kubernetes installation and worker onboarding.
- Cross-host Pod networking using Tailscale and Flannel.
- Parameterized Ansible playbooks for targeted node operations.
- GitHub Actions integration for repeatable infrastructure validation.

## Future Improvements

- Multi-control-plane Kubernetes high availability.
- Automated configuration drift detection.
- Expanded infrastructure testing and health checks.
