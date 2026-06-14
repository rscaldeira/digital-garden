---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/tests/PVC - Basic Test.yaml/","dg-note-properties":{}}
---

# PVC - Basic Test

## Objetivo

Validar provisionamento básico de volume Portworx em pod simples.

## StorageClass sugerida

Usar `px-app-repl1` para teste rápido ou `px-app-repl2` para validação com HA básico.

## YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: basic-pvc-test
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: px-app-repl1
---
apiVersion: v1
kind: Pod
metadata:
  name: basic-pvc-pod
spec:
  containers:
    - name: app
      image: registry.k8s.io/busybox
      command: ["/bin/sh", "-c", "echo portworx-ok > /data/test.txt && sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: basic-pvc-test
```

## Aplicar

```bash
oc apply -f basic-test.yaml
```

## Validar

```bash
oc get pvc
oc get pod basic-pvc-pod
oc exec -it basic-pvc-pod -- cat /data/test.txt
```

## Resultado esperado

* PVC em estado `Bound`
* Pod em estado `Running`
* Arquivo gravado e lido com sucesso
