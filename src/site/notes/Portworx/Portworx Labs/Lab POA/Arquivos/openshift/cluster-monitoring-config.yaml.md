---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/openshift/cluster-monitoring-config.yaml/","dg-note-properties":{}}
---


# OpenShift - cluster-monitoring-config

Arquivo real:

```text
cluster-monitoring-config.yaml
```

Objetivo:

Habilitar monitoramento de workloads/projetos de usuário no OpenShift antes da instalação do Portworx.

Aplicar antes de instalar o Portworx Operator.

Uso:

```bash
oc apply -f cluster-monitoring-config.yaml
```

Manifest:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
```

Validação:

```bash
oc get configmap cluster-monitoring-config -n openshift-monitoring
oc get pods -n openshift-user-workload-monitoring
```