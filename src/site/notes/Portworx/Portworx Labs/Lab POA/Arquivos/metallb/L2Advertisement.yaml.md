---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/metallb/L2Advertisement.yaml/","dg-note-properties":{}}
---


# MetalLB - L2Advertisement

  
Arquivo real:
 
```text
l2advertisement.yaml
```

Uso:

```bash
oc apply -f l2advertisement.yaml
```

Manifest:

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: lab-poa-l2
  namespace: metallb-system
spec:
  ipAddressPools:
    - lab-poa-pool
```