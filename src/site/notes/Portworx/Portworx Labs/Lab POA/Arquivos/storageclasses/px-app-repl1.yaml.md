---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl1.yaml/","dg-note-properties":{}}
---

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-app-repl1
provisioner: pxd.portworx.com
parameters:
  repl: "1"
  io_profile: "auto"
  priority_io: "medium"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
```