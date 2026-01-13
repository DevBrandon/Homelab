## Homelab Raspberry Pi Provisioning – Context & Plan

### Background

I am a professional software engineer with strong knowledge of:

* Linux systems
* Networking
* Distributed systems
* Containers and automation

This project manages a small fleet of **Raspberry Pi devices** used as a homelab.
The primary goals are:

* Fast, repeatable setup
* Easy maintenance and updates
* Minimal re-imaging
* Declarative, idempotent configuration

---

### Core Philosophy

* **Avoid custom OS images unless strictly necessary**
* Prefer **configuration management over image baking**
* Treat hosts as **mostly immutable**
* Everything above the OS runs in **containers**
* Configuration lives in **Git as source of truth**

---

### Base OS

* Use **stock Ubuntu Server LTS (arm64)** or **Raspberry Pi OS Lite (64-bit)**
* Manual or cloud-init bootstrap only:

  * Enable SSH
  * Create user
  * Install Python
  * Add SSH keys

No applications or services are baked into the image beyond what's required to run Ansible.

---

### Provisioning Model (Two-Phase)

#### Phase 1: Minimal Bootstrap

* Manual install or cloud-init
* Responsibilities:

  * Hostname
  * SSH access
  * Python installed
  * Optional kernel cgroup settings

#### Phase 2: Ansible (Authoritative)

Ansible is the **single source of truth**.

It is responsible for:

* System updates
* Host hardening
* Docker installation
* Service deployment
* Ongoing maintenance

Re-imaging should be rare.

---

### Configuration Management (Ansible)

#### Structure

```
ansible/
├── inventories/
│   ├── prod/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   │       └── all.yml
├── roles/
│   ├── common/
│   ├── docker/
│   ├── portainer/
│   ├── monitoring/
│   └── k3s/ (optional)
├── playbooks/
│   ├── bootstrap.yml
│   └── site.yml
```

#### Principles

* Idempotent roles
* Role-based host grouping (by function, not device)
* Use community roles where appropriate (e.g. geerlingguy.docker)

---

### Container Strategy

* Docker installed and managed via Ansible
* All applications run as containers
* Prefer Docker Compose or Ansible docker modules
* Portainer used for:

  * Visibility
  * Stack management
  * Debugging

Avoid installing applications directly on the host.

---

### Inventory Model

Hosts are grouped by **purpose**, not by name.

Example:

```yaml
all:
  children:
    infra:
      hosts:
        pi-1:
        pi-2:
    media:
      hosts:
        pi-3:
```

Roles are applied based on group membership.

---

### Kubernetes (Optional)

* Consider **k3s** only if:

  * HA is required
  * Declarative workloads provide real benefit
* Default approach is **Docker + Compose**
* Kubernetes is not the baseline

---

### Custom Image Policy

Custom images (via Packer) are **not the default**.

Allowed only if:

* Provisioning many Pis frequently
* Offline installs are required
* Bootstrapping time becomes a real bottleneck

Even then:

* Images should contain only OS updates + Python (+ maybe Docker)
* Ansible still performs final configuration

---

### End Goal

A Raspberry Pi can be:

1. Flashed with a stock OS
2. Booted and reachable via SSH
3. Fully configured by running:

```bash
ansible-playbook site.yml
```

No manual setup beyond bootstrap.

---

### Expectations for This ChatGPT Instance

When assisting with this project:

* Assume strong technical background
* Favor maintainable, production-style patterns
* Prefer declarative, Git-backed solutions
* Avoid "beginner" explanations unless explicitly requested
* Optimize for long-term maintainability over cleverness
