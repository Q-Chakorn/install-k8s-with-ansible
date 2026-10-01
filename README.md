# Kubernetes Cluster with Ansible

Ansible playbook สำหรับติดตั้ง Kubernetes cluster ด้วย kubeadm

## Requirements

- Ubuntu 20.04/22.04 LTS (target nodes)
- Ansible 2.9+
- SSH access ไปยัง nodes ด้วย sudo privilege

## โครงสร้างไฟล์

```
k8s-cluster/
├── ansible.cfg
├── site.yml                 # Main playbook
├── inventory/
│   └── hosts.yml            # กำหนด master/worker nodes
├── group_vars/
│   └── all.yml              # ตัวแปร global
└── roles/
    ├── common/              # เตรียม system
    ├── containerd/          # ติดตั้ง containerd
    └── kubernetes/          # ติดตั้ง kubeadm, kubelet, kubectl
```

## วิธีใช้งาน

### 1. แก้ไข Inventory

แก้ไขไฟล์ `inventory/hosts.yml`:

```yaml
masters:
  hosts:
    master1:
      ansible_host: <MASTER_IP>

workers:
  hosts:
    worker1:
      ansible_host: <WORKER1_IP>
    worker2:
      ansible_host: <WORKER2_IP>
```

### 2. แก้ไขตัวแปร (ถ้าต้องการ)

แก้ไขไฟล์ `group_vars/all.yml`:

```yaml
kubernetes_version: "1.30"
cni_plugin: calico  # หรือ flannel
pod_network_cidr: "10.244.0.0/16"
```

### 3. รัน Playbook

```bash
# ทดสอบ connection
ansible all -m ping

# รัน playbook
ansible-playbook site.yml

# หรือรันแบบ verbose
ansible-playbook site.yml -v
```

### 4. ตรวจสอบ Cluster

บน master node:

```bash
kubectl get nodes
kubectl get pods -A
```

## Playbooks เสริม

### Reset Cluster

สร้างไฟล์ `reset.yml`:

```yaml
---
- hosts: k8s_cluster
  become: true
  tasks:
    - name: Reset kubeadm
      command: kubeadm reset -f
      ignore_errors: yes
```

รัน: `ansible-playbook reset.yml`

## หมายเหตุ

- Playbook นี้ใช้ได้กับ Ubuntu (Debian-based)
- ต้องการ RAM อย่างน้อย 2GB ต่อ node
- Master node ต้องการอย่างน้อย 2 CPU cores
