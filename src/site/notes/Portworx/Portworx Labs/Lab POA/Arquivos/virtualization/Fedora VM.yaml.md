---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Fedora VM.yaml/","dg-note-properties":{}}
---

# Fedora VM

## Objetivo

Criar uma VM Fedora simples em OpenShift Virtualization usando Portworx como storage principal.

## StorageClass sugerida

Usar `px-vm-block-repl2` como classe principal para o disco da VM.

## YAML

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: fedora-vm
  namespace: vms
spec:
  running: true
  dataVolumeTemplates:
    - metadata:
        name: fedora-rootdisk
        annotations:
          cdi.kubevirt.io/storage.usePopulator: "false"
      spec:
        source:
          registry:
            url: docker://quay.io/containerdisks/fedora:latest
            pullMethod: node
        storage:
          accessModes:
            - ReadWriteMany
          resources:
            requests:
              storage: 30Gi
          storageClassName: px-vm-block-repl2
          volumeMode: Block
  template:
    metadata:
      labels:
        kubevirt.io/domain: fedora-vm
      annotations:
        kubevirt.io/allow-pod-bridge-network-live-migration: ""
    spec:
      domain:
        cpu:
          cores: 2
        resources:
          requests:
            memory: 4Gi
        devices:
          disks:
            - name: rootdisk
              bootOrder: 1
              disk:
                bus: virtio
            - name: cloudinitdisk
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
            name: fedora-rootdisk
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |
              #cloud-config
              user: fedora
              password: fedora
              chpasswd: { expire: False }
```

## Aplicar

```bash
oc apply -f fedora-vm.yaml
```

## Validar

```bash
oc get vm -n vms
oc get vmi -n vms
oc get dv -n vms
```

## Resultado esperado

* VM criada com sucesso
* DataVolume criado e em progresso ou concluído
* VMI em estado `Running`
