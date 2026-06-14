---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/metallb/IPAddressPool.yaml/","dg-note-properties":{}}
---


# MetalLB - IPAddressPool

  

Arquivo real:
 

```text
ipaddresspool.yaml
```
  

Uso:
  

```bash
oc apply -f ipaddresspool.yaml
```


Manifest:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: lab-poa-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.X.200-192.168.X.220
```