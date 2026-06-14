---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Windows VM.yaml/","dg-note-properties":{}}
---

# Windows VM

## Objetivo

Criar uma VM Windows para lab usando Portworx RWX Block.

## Observação

Este exemplo considera:

* disco `C:` vindo de um `DataVolume` importado pelo CDI
* disco `D:` vindo de um PVC adicional já criado

## Pré-requisitos

* `DataVolume` chamado `windows-os-disk`
* PVC chamado `win-data-disk-d`
* StorageClass `px-vm-block-repl2`

## YAML

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: windows-vm
  namespace: vms
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/domain: windows-vm
      annotations:
        kubevirt.io/allow-pod-bridge-network-live-migration: ""
    spec:
      domain:
        cpu:
          cores: 4
        resources:
          requests:
            memory: 8Gi
        devices:
          disks:
            - name: rootdisk
              bootOrder: 1
              disk:
                bus: virtio
            - name: datadisk
              disk:
                bus: virtio
          interfaces:
            - name: default
              masquerade: {}
        firmware:
          bootloader:
            efi: {}
      networks:
        - name: default
          pod: {}
      volumes:
        - name: rootdisk
          dataVolume:
            name: windows-os-disk
        - name: datadisk
          persistentVolumeClaim:
            claimName: win-data-disk-d
```

## Aplicar

```bash
oc apply -f windows-vm.yaml
```

## Validar

```bash
oc get vm -n vms
oc get vmi -n vms
oc describe vm windows-vm -n vms
```

## Resultado esperado

* VM criada com sucesso
* Disco `C:` anexado via `DataVolume`
* Disco `D:` anexado via PVC adicional
* VM pronta para boot manual ou automático
