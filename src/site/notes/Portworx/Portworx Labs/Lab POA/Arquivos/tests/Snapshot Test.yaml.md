---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/tests/Snapshot Test.yaml/","dg-note-properties":{}}
---

# Snapshot Test

## Objetivo

Validar snapshot e restore de PVC em FlashArray Direct Access com Portworx CSI.

## YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: snap-source-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: px-app-repl2
---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: px-fa-direct-access-snapshotclass
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: pxd.portworx.com
deletionPolicy: Delete
---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: snap-source-volumesnapshot
spec:
  volumeSnapshotClassName: px-fa-direct-access-snapshotclass
  source:
    persistentVolumeClaimName: snap-source-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: snap-restore-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: px-app-repl2
  dataSource:
    kind: VolumeSnapshot
    name: snap-source-volumesnapshot
    apiGroup: snapshot.storage.k8s.io
```

## Aplicar

```bash
oc apply -f snapshot-test.yaml
```

## Validar

```bash
oc get pvc
oc get volumesnapshot
```

## Resultado esperado

* `VolumeSnapshot` criado com sucesso
* PVC de restore em estado `Bound`
* Novo PVC criado a partir do snapshot
