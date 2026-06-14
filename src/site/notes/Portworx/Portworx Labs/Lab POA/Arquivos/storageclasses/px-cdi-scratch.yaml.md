---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-cdi-scratch.yaml/","dg-note-properties":{}}
---


```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-cdi-scratch
provisioner: pxd.portworx.com
parameters:
  repl: "1"
  sharedv4: "true"
  sharedv4_mount_options: vers=3.0,nolock
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
```
