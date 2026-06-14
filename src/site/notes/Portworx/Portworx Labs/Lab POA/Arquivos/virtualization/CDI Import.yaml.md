---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/CDI Import.yaml/","dg-note-properties":{}}
---

# CDI Import

## Objetivo

Importar imagem de sistema operacional via CDI para uso em VMs no OpenShift Virtualization.

## Observação

Este arquivo traz um exemplo genérico para:

* importar disco de sistema via `DataVolume`
* criar disco adicional de dados para Windows

## YAML

```yaml
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: windows-os-disk
  namespace: vms
  annotations:
    cdi.kubevirt.io/storage.usePopulator: "false"
spec:
  source:
    http:
      url: "https://URL-DA-IMAGEM/windows-server.qcow2"
  storage:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 128Gi
    storageClassName: px-vm-block-repl2
    volumeMode: Block
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: win-data-disk-d
  namespace: vms
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
---
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: fedora-os-disk
  namespace: vms
  annotations:
    cdi.kubevirt.io/storage.usePopulator: "false"
spec:
  source:
    http:
      url: "https://URL-DA-IMAGEM/fedora.qcow2"
  storage:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 40Gi
    storageClassName: px-vm-block-repl2
    volumeMode: Block
```

## Aplicar

```bash
oc apply -f cdi-import.yaml
```

## Validar

```bash
oc get dv -n vms
oc get pvc -n vms
```

## Resultado esperado

* DataVolumes criados com sucesso
* PVCs em criação ou `Bound`
* Imagens prontas para consumo pelas VMs
