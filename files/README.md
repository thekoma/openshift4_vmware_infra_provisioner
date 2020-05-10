# To run the installation please fill the variables

### Check Installation status:
```
{{playbook_dir}}/tmp/bin/openshift-install --dir={{playbook_dir}}/tmp/cluster_conf wait-for bootstrap-complete  --log-level=info  "
```

### To configure the OC client export this variable:  '
```
export KUBECONFIG={{playbook_dir}}/tmp/cluster_conf/auth/kubeconfig  "
/usr/bin/oc login -h api.{{ocp_installer.clustername}}{{ocp_installer.basedomain}} -u kubeadmin -p {{ lookup('file', playbook_dir + '/tmp/cluster_conf/auth/kubeadmin-password') }}  "
```

### To approve the pending nodes you should run this (you may need to do this multiple times):  "
```
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```
### To configure the registry for non production cluster:'
```
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
```
