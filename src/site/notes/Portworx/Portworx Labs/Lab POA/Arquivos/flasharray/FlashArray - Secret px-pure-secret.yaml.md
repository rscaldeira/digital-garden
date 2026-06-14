---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - Secret px-pure-secret.yaml/","dg-note-properties":{}}
---

# FlashArray - Secret px-pure-secret

## Objetivo

Criar o secret `px-pure-secret` no namespace do Portworx para disponibilizar o arquivo `pure.json` durante a instalação ou operação do cluster.

## YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: px-pure-secret
  namespace: portworx
type: Opaque
stringData:
  pure.json: |
    {
      "FlashArrays": [
        {
          "MgmtEndPoint": "fa-mgmt.lab.local",
          "APIToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
      ]
    }
```

## Ajustes comuns

* Trocar `namespace: portworx` se seu StorageCluster estiver em outro namespace
* Substituir `MgmtEndPoint` pelo endpoint de management do FlashArray
* Substituir `APIToken` pelo token gerado no array
* Se usar secure multi-tenancy, adicionar `Realm` dentro do JSON

## Aplicar

```bash
oc apply -f px-pure-secret.yaml
```

## Validar

```bash
oc get secret -n portworx px-pure-secret
oc describe secret -n portworx px-pure-secret
```

## Alternativa por linha de comando

```bash
oc create secret generic px-pure-secret \
  --namespace portworx \
  --from-file=pure.json=./pure.json
```
