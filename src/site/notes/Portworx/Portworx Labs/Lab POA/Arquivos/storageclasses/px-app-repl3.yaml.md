---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl3.yaml/","dg-note-properties":{}}
---


```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-app-repl3
provisioner: pxd.portworx.com
parameters:
  repl: "3"
  io_profile: "auto"
  priority_io: "high"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
```