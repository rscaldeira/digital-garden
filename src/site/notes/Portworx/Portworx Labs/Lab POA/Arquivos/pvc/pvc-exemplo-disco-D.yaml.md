---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/pvc/pvc-exemplo-disco-D.yaml/","dg-note-properties":{}}
---


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: win-data-disk-d
  labels:
    portworx.io/app: kubevirt
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: px-vm-block-repl2
  volumeMode: Block
```