---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/pvc/pvc-DataVolume-para-disco-C.yaml/","dg-note-properties":{}}
---


```yaml
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: win-os-disk-c
spec:
  source:
    blank: {}
  pvc:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 128Gi
    storageClassName: px-vm-block-repl2
    volumeMode: Block
```