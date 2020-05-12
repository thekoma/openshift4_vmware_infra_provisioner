Openshift 4 vmware infra provisioner
=========

This Role provides the simplest way to install openshift 4 taking care of all the network configuration but DNS.

Features:
------------
- **You don't need DHCP**
- check for yout environment
- Validate your configuration
- Validate your DNS
- Configure an HaProxy (not deploy it, only configure ATM)
- Configure an Nginx web server (not deploy it, only configure ATM)
- Build the iPXE iso image
- Create the VM
- Boot the vm injecting network configuration
- Install the cluster

TODO:
------------
- ~~Check for yout environment~~
- ~~Validate your configuration~~
- ~~Validate your DNS~~
- ~~Configure an HaProxy (not deploy it, only configure ATM)~~
- ~~Configure an Nginx web server (not deploy it, only configure ATM)~~
- ~~Build the iPXE iso image~~
- ~~Create the VM~~
- ~~Boot the vm injecting network configuration~~
- ~~Install the cluster~~
- Validate minimal requirements
- Deploy VM for HaProxy and NginX
- Build HA/Vrrp for HaProxy/NginX

Requirements
------------

A Bastion node where execute thos tasks and where install a temporary webserver
Rhel/Centos 8
Ansible 2.9

Role Variables
--------------

```bash
# Please lookup the variables in 
echo defaults/main.yml
```

Dependencies
------------

Tested on RHEL8 and Centos8
Tested with Ansible 2.9

Example Playbook
----------------

```yaml
---
- hosts: all
  remote_user: root
  roles:
    - openshift4_infra_provisioner
```

Example Inventory
----------------

```yaml
---
haproxy:
  hosts:
    bastion.ocp.okd01.lab.local
webserver:
  hosts:
    bastion.ocp.okd01.lab.local
all:
    vars:
      enable_reservation: False
      dns_fail_is_not_fatal: False
      ptr_fail_is_not_fatal: True
      checks_only: False
      #... look for the entire set of variables in defaults/main.yml ...
```

License
-------

MIT

Author Information
------------------

# To run the installation please fill the variables

### Check Installation status:
```bash
openshift-install \
  --dir={{playbook_dir}}/tmp/cluster_conf wait-for bootstrap-complete \
  --log-level=info
```

### To configure the OC client export this variable:  '
```bash
export KUBECONFIG={{playbook_dir}}/tmp/cluster_conf/auth/kubeconfig

oc login \
  -h api.{{ocp_installer.clustername}}{{ocp_installer.basedomain}} \
  -u kubeadmin \
  -p {{ lookup('file', playbook_dir + '/tmp/cluster_conf/auth/kubeadmin-password') }}
```

### To approve the pending nodes you should run this (you may need to do this multiple times):  "
```bash
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```
### To configure the registry for non production cluster:'
```bash
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'`
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
```
