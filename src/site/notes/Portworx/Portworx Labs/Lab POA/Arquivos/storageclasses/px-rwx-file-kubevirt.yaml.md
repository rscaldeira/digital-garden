---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-rwx-file-kubevirt.yaml/","dg-note-properties":{}}
---


```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-rwx-file-kubevirt
provisioner: pxd.portworx.com
parameters:
  repl: "2"
  sharedv4: "true"
  sharedv4_mount_options: vers=3.0,nolock
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```