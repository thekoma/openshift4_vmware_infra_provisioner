Openshift 4 vmware infra provisioner
=========

This Role provides the simplest way to install openshift 4 taking care of all the network configuration but DNS.

Requirements
------------

A Bastion node where execute thos tasks and where install a temporary webserver

Role Variables
--------------

```yaml
---

ocp_vm_cleanup: no             # Set it to True to cleanup the environment
ocp_vm_create_webserver: yes  # Enable the creation of the webserver.
ocp_vm_create_haproxy: yes
ocp_vm_create_ipxe: yes

vcenter:
    host: vcenter.vsphere.local
    user: administrator@vsphere.local
    password: password
    datacenter: datacenter01
    storage: /OKD
    folder: /OKD
    validate_certs: no
    datastore: "datastore01" 

ocp_installer:
    basedomain: lab.local
    clustername: okd01
    pullsecret: FILLME 
    sshkey: FILLME
    kernel: https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-4.4.3-x86_64-installer-kernel-x86_64
    initrd: https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-4.4.3-x86_64-installer-initramfs.x86_64.img
    rhcosimage: https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-4.4.3-x86_64-metal.x86_64.raw.gz
    installer: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
    clients: https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz

hosts_defaults:
    boot: iso
    iso: "ISOs/ocp_pxe.iso"
    network:
      name: MyNetWork
      mask: 255.255.255.0
      gw: 192.168.0.1
      dns:
          - 192.168.0.2
          - 192.168.0.3
    generic:
      memory_mb: 16384
      num_cpus: 4
      num_cpu_cores_per_socket: 1
      mem_limit: 16384
      mem_reservation: 0
      disk: 120
      memory_reservation_lock: False
      guest_id: other4xLinux64Guest
      ignition: none
    master:
      memory_mb: 16384
      num_cpus: 4
      num_cpu_cores_per_socket: 1
      mem_limit: 16384
      mem_reservation: 0
      disk: 120
      memory_reservation_lock: False
      guest_id: coreos64Guest
      ignition: master.ign
    bootstrap:
      memory_mb: 16384
      num_cpus: 4
      num_cpu_cores_per_socket: 1
      mem_limit: 16384
      mem_reservation: 0
      disk: 120
      memory_reservation_lock: False
      guest_id: coreos64Guest
      ignition: bootstrap.ign
    worker:
      memory_mb: 8192
      num_cpus: 4
      num_cpu_cores_per_socket: 1
      mem_limit: 8192
      mem_reservation: 0
      disk: 120
      memory_reservation_lock: False
      guest_id: coreos64Guest
      ignition: worker.ign

vm_hosts:
    - hostname: master1.ocp.okd01.lab.local
      type: master
      network:
          ip: 192.168.168.180 # Required
          name: MyNetWork     # Optional
          mask: 255.255.255.0 # Optional
          gw: 192.168.0.1     # Optional
          dns:                # Optional
              - 192.168.0.2   # Optional
              - 192.168.0.3  # Optional
    - hostname: master2.ocp.okd01.lab.local
      type: master
      network:
          ip: 192.168.168.181
    - hostname: master3.ocp.okd01.lab.local
      type: master
      network:
          ip: 192.168.168.182
    - hostname: worker1.ocp.okd01.lab.local
      type: worker
      network:
          ip: 192.168.168.183
    - hostname: worker2.ocp.okd01.lab.local
      type: worker
      network:
          ip: 192.168.168.184
    - hostname: worker3.ocp.okd01.lab.local
      type: worker
      network:
          ip: 192.168.168.185
    - hostname: bootstrap.ocp.okd01.lab.local
      type: bootstrap
      network:
          ip: 192.168.168.187
```

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

```yaml
---
- hosts: localhost
  remote_user: root
  roles:
    - openshift4_infra_provisioner
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
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' |\
  xargs oc adm certificate approve
```
### To configure the registry for non production cluster:'
```bash
oc patch configs.imageregistry.operator.openshift.io \
  cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
```
