---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl2.yaml/","dg-note-properties":{}}
---


```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-app-repl2
provisioner: pxd.portworx.com
parameters:
  repl: "2"
  io_profile: "auto"
  priority_io: "high"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
```