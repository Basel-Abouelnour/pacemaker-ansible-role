
# pacemaker-ansible-role

## Overview

This Ansible role deploys a **Pacemaker HA cluster** on CentOS 9 stream environment with 2 nodes.  
Pacemaker is a **high-availability cluster manager** that monitors resources and automatically moves them to healthy nodes in case of failure.

In this role, **Apache HTTP Server (`httpd`)** is used as an example resource, colocated with a **Virtual IP (VIP)** to provide a simple HA web service in an active / passive setup.

---

## Installed Packages and Purpose

| Package            | Purpose                                                                          |
| ------------------ | -------------------------------------------------------------------------------- |
| `pacemaker`        | Core cluster manager to monitor and manage resources                             |
| `corosync`         | Messaging layer between cluster nodes for quorum and communication               |
| `pcs`              | Command-line tool to configure and manage Pacemaker clusters                     |
| `resource-agents`  | Collection of scripts to manage various resources (Apache, IPs, databases, etc.) |
| `fence-agents-all` | Tools for node fencing (disabled in this demo)                                    |
| `httpd`            | Apache web server used as a cluster resource                                     |

> **Note:** Fencing (`STONITH`) is disabled in this demo because there are no dedicated fencing devices and this is a lab environment. In production, fencing is critical to prevent split-brain scenarios by applying (`STONITH`) which is `Shooting The Other Node In The Head`, AKA powering it off.

---

## Inventory Example

```ini
[all]
target-1
target-2

[targets]
target-1
target-2
````

---

## Deploy the Cluster

**Playbook example:**

```yaml
- hosts: targets
  become: yes
  roles:
    - pacemaker
```

**Run the Playbook:**

```bash
ansible-playbook deploy-cluster.yml -i hosts.ini
# test the results
curl http://VIP
```

* This will:

  1. Install & enable Pacemaker requirements on both targets.
  2. Configure a two-node cluster with a VIP.
  3. Create `httpd` as a managed resource colocated with the VIP.
---

## Delete the Cluster

**Playbook:**

```yaml
- hosts: targets
  become: true
  gather_facts: false
  tasks:
    - name: Stop cluster
      command: pcs cluster stop --all
      run_once: true
      failed_when: false

    - name: Disable cluster at boot
      command: pcs cluster disable --all
      run_once: true
      failed_when: false

    - name: Destroy cluster configuration
      command: pcs cluster destroy --force
      run_once: true
      failed_when: false
```

**Run the Playbook:**

```bash
ansible-playbook delete-cluster.yml -i hosts.ini
```

* This will stop all cluster services, disable autostart, and remove cluster configuration.

---

## Failure Test Playbook

**Purpose:** Stop `httpd` on `target-1` to validate that Pacemaker moves the resource to `target-2`.

**Playbook (`test-httpd-failover.yml`):**

```yaml
- name: Test Apache Failover in Pacemaker Cluster
  hosts: target-1
  become: yes
  gather_facts: yes
  tasks:

    - name: Stop httpd on target-1 to simulate failure
      service:
        name: httpd
        state: stopped

    - name: Wait for cluster to detect failure
      pause:
        seconds: 15
```

**Run the Playbook:**

```bash
ansible-playbook test-failover.yml -i hosts.ini
# test the results again
# notice that the resource moved to the second node
curl http://VIP
```

---

## Role Usage

> This role is published on Ansible-Galaxy and can be installed using:

```bash
ansible-galaxy role install Basel-Abouelnour.Pacemaker-Cluster-Installation
```

---



