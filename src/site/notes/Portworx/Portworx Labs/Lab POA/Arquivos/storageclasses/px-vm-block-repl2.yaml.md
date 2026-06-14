---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-vm-block-repl2.yaml/","dg-note-properties":{}}
---


```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-vm-block-repl2
  annotations:
    storageclass.kubevirt.io/is-default-virt-class: "true"
provisioner: pxd.portworx.com
parameters:
  repl: "2"
  io_profile: "db_remote"
  priority_io: "high"
  nodiscard: "true"
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```
