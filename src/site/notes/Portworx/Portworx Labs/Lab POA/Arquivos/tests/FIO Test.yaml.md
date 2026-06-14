---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/tests/FIO Test.yaml/","dg-note-properties":{}}
---

# FIO Test

## Objetivo

Executar benchmark simples de FIO em PVC Portworx.

## StorageClass sugerida

* `px-app-repl1` para baseline simples
* `px-app-repl2` para comparação com HA básico
* `px-app-repl3` para demonstração de máxima proteção

## YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fio-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: px-app-repl2
---
apiVersion: v1
kind: Pod
metadata:
  name: fio-test
spec:
  restartPolicy: Never
  containers:
    - name: fio
      image: xridge/fio:latest
      command:
        - fio
        - --name=randwrite
        - --filename=/data/fio-test-file
        - --size=2G
        - --bs=4k
        - --rw=randwrite
        - --iodepth=32
        - --ioengine=libaio
        - --direct=1
        - --numjobs=1
        - --runtime=60
        - --time_based
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: fio-pvc
```

## Aplicar

```bash
oc apply -f fio-test.yaml
```

## Validar

```bash
oc get pod fio-test
oc logs fio-test
```

## Resultado esperado

* PVC em estado `Bound`
* Pod finaliza sem erro
* Resultado de IOPS, bandwidth e latency disponível no log
