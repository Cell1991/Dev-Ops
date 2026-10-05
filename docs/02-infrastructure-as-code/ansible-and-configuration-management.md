# Track 02 // Ansible & Immutable Configuration Management

While cloud-native workloads run in containers, foundational Virtual Machines, bastion hosts, and on-prem hypervisors require idempotent configuration management.

---

## 1. Idempotency & Role-Based Architecture

Ansible operates over agentless SSH with push-based execution:

```yaml
---
# playbook.yml - Enterprise Node Hardening
- name: Harden Production Kubernetes Worker Nodes
  hosts: k8s_workers
  become: true
  gather_facts: true

  vars:
    sysctl_k8s_params:
      net.bridge.bridge-nf-call-iptables: 1
      net.ipv4.ip_forward: 1
      vm.max_map_count: 262144

  tasks:
    - name: Ensure sysctl networking parameters are persisted
      ansible.posix.sysctl:
        name: "{{ item.key }}"
        value: "{{ item.value }}"
        state: present
        reload: true
      loop: "{{ sysctl_k8s_params | dict2items }}"

    - name: Install containerd runtime packages
      ansible.builtin.apt:
        name:
          - containerd.io
          - ipset
        state: present
        update_cache: true

    - name: Configure containerd systemd cgroup driver
      ansible.builtin.copy:
        dest: /etc/containerd/config.toml
        content: |
          version = 2
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
        mode: '0644'
      notify: Restart containerd

  handlers:
    - name: Restart containerd
      ansible.builtin.systemd:
        name: containerd
        state: restarted
        enabled: true
```
