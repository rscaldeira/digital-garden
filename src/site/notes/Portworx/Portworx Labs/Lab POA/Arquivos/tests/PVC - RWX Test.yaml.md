---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/tests/PVC - RWX Test.yaml/","dg-note-properties":{}}
---

# PVC - RWX Test

## Objetivo

Validar acesso compartilhado RWX em filesystem.

## Observação

Para teste de RWX em pod, usar shared filesystem.
Para disco principal de VM, continuar usando RWX Block para KubeVirt.

## YAML

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-rwx-file-repl2
provisioner: pxd.portworx.com
parameters:
  repl: "2"
  sharedv4: "true"
  sharedv4_mount_options: vers=3.0,nolock
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rwx-pvc-test
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: px-rwx-file-repl2
---
apiVersion: v1
kind: Pod
metadata:
  name: rwx-pod-1
spec:
  containers:
    - name: app
      image: registry.k8s.io/busybox
      command: ["/bin/sh", "-c", "echo shared-data > /data/shared.txt && sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: rwx-pvc-test
---
apiVersion: v1
kind: Pod
metadata:
  name: rwx-pod-2
spec:
  containers:
    - name: app
      image: registry.k8s.io/busybox
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: rwx-pvc-test
```

## Aplicar

```bash
oc apply -f rwx-test.yaml
```

## Validar

```bash
oc get pvc
oc get pod rwx-pod-1 rwx-pod-2
oc exec -it rwx-pod-2 -- cat /data/shared.txt
```

## Resultado esperado

* PVC RWX em estado `Bound`
* Ambos os pods em estado `Running`
* O segundo pod consegue ler o arquivo criado pelo primeiro
