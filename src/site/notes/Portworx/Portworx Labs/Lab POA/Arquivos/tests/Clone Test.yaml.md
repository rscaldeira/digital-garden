---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/tests/Clone Test.yaml/","dg-note-properties":{}}
---

# Clone Test

## Objetivo

Validar clone de PVC usando `dataSource.kind: PersistentVolumeClaim`.

## YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: clone-source-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: px-app-repl2
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: clone-target-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: px-app-repl2
  dataSource:
    kind: PersistentVolumeClaim
    name: clone-source-pvc
```

## Aplicar

```bash
oc apply -f clone-test.yaml
```

## Validar

```bash
oc get pvc
```

## Resultado esperado

* PVC de origem em estado `Bound`
* PVC de clone em estado `Bound`
* Clone criado sem alterar o volume original
