---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/12 - Troubleshooting/","dg-note-properties":{}}
---


# Troubleshooting

## Objetivo

Centralizar procedimentos de troubleshooting do Lab POA.

Este documento deve ser utilizado durante:

- Instalação
- Validação
- Operação
- Demonstrações
- Reconstruções futuras do ambiente

Sempre registrar:

- Sintoma
- Causa
- Solução
- Referência
- Data

---

# Referências do Projeto

Instalação:

- [[Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift\|05 - Instalação OpenShift]]
- [[Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB\|06 - Instalação MetalLB]]
- [[Portworx/Portworx Labs/Lab POA/07 - Instalação Portworx\|07 - Instalação Portworx]]
- [[Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization\|08 - OpenShift Virtualization]]
- [[Portworx/Portworx Labs/Lab POA/09 - PX-Central\|09 - PX-Central]]
- [[Portworx/Portworx Labs/Lab POA/10 - PX-Backup\|10 - PX-Backup]]

Validações:

- [[Portworx/Portworx Labs/Lab POA/11 - Testes de Validação\|11 - Testes de Validação]]

---

# OpenShift

## Cluster Operators Degraded

### Sintoma

```bash
oc get co
```

Algum Cluster Operator apresenta:

- Degraded=True
- Available=False

### Verificações

```bash
oc get co
```

```bash
oc describe co <operator>
```

```bash
oc get events -A --sort-by='.lastTimestamp'
```

### Possíveis Causas

- DNS
- Recursos insuficientes
- Problemas de rede
- Problemas de storage

---

## Node NotReady

### Sintoma

```bash
oc get nodes
```

Node em estado NotReady.

### Verificações

```bash
oc describe node <node>
```

```bash
oc get events --sort-by='.lastTimestamp'
```

### Possíveis Causas

- Rede
- MCP degradado
- Falha no host
- Storage

---

## MCP Degraded

### Sintoma

```bash
oc get mcp
```

### Verificações

```bash
oc describe mcp
```

```bash
oc get machineconfig
```

### Possíveis Causas

- MachineConfig inválida
- Reboot incompleto
- Configuração incorreta

---

# MetalLB

## Service sem EXTERNAL-IP

### Sintoma

```bash
oc get svc
```

EXTERNAL-IP pendente.

### Verificações

```bash
oc get metallb -A
```

```bash
oc get ipaddresspool -A
```

```bash
oc get l2advertisement -A
```

### Possíveis Causas

- Pool inexistente
- Pool esgotado
- L2Advertisement ausente
- Configuração inválida

---

## IP recebido mas sem acesso

### Sintoma

Service recebe EXTERNAL-IP mas não responde.

### Verificações

```bash
arping <external-ip>
```

```bash
oc describe svc <service>
```

### Possíveis Causas

- Firewall
- Gateway incorreto
- VLAN incorreta
- IP duplicado

---

# FlashArray

## Falha de comunicação com FlashArray

### Sintoma

PVCs não provisionam.

### Verificações

- Endpoint configurado
- API Token
- Reachability

### Possíveis Causas

- Endpoint incorreto
- Token inválido
- Firewall

---

## px-pure-secret inválido

### Sintoma

Provisionamento falha.

### Verificações

```bash
oc get secret px-pure-secret -A
```

### Possíveis Causas

- pure.json incorreto
- Endpoint incorreto
- API Token inválido

### Referências

- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - pure.json\|Arquivos/flasharray/FlashArray - pure.json]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - Secret px-pure-secret.yaml\|Arquivos/flasharray/FlashArray - Secret px-pure-secret.yaml]]

---

# Portworx

## Pods não iniciam

### Sintoma

Pods em:

- Pending
- CrashLoopBackOff

### Verificações

```bash
oc get pods -n portworx
```

```bash
oc logs <pod> -n portworx
```

### Possíveis Causas

- StorageCluster inválido
- FlashArray inacessível
- Secret inválido

---

## StorageCluster não fica Online

### Sintoma

```bash
oc get storagecluster -A
```

### Verificações

```bash
oc describe storagecluster
```

```bash
oc get events -A
```

### Possíveis Causas

- FlashArray
- Rede
- Metadata Device
- Cloud Drives

---

## Cluster não forma

### Sintoma

```bash
pxctl status
```

Nodes Offline.

### Verificações

```bash
pxctl cluster list
```

### Possíveis Causas

- Firewall
- Rede
- Descoberta entre nodes

---

## PVC fica Pending

### Sintoma

```bash
oc get pvc
```

PVC permanece Pending.

### Verificações

```bash
oc describe pvc <pvc>
```

### Possíveis Causas

- StorageClass inválida
- Pool indisponível
- FlashArray inacessível

---

# Fibre Channel

## LUNs não aparecem

### Verificações

```bash
systool -c fc_host -v
```

```bash
multipath -ll
```

### Possíveis Causas

- Zoning incorreto
- HBA offline
- FlashArray

---

## Multipath incorreto

### Verificações

```bash
multipath -ll
```

### Possíveis Causas

- MachineConfig incorreta
- Multipath não aplicado

---

# OpenShift Virtualization

## VM não inicia

### Verificações

```bash
oc get vm
```

```bash
oc get vmi
```

```bash
oc describe vm <vm>
```

### Possíveis Causas

- Storage indisponível
- Recursos insuficientes
- Imagem inválida

---

## CDI Import falha

### Verificações

```bash
oc get dv
```

```bash
oc get pvc
```

### Possíveis Causas

- StorageClass incorreta
- URL inválida
- Espaço insuficiente

### Referências

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/CDI Import.yaml\|Arquivos/virtualization/CDI Import.yaml]]

---

## Live Migration falha

### Verificações

Eventos da VM.

```bash
oc describe vm <vm>
```

### Possíveis Causas

- Recursos insuficientes
- Limitação da StorageClass
- Rede
- Configuração KubeVirt

### Referência

- [[Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization\|08 - OpenShift Virtualization]]

---

# PX-Central

## Dashboard inacessível

### Verificações

```bash
oc get svc -A
```

```bash
oc get pods -A | grep px
```

### Possíveis Causas

- Service sem IP
- MetalLB
- Pod indisponível

---

## Cluster não aparece

### Possíveis Causas

- Portworx Offline
- Comunicação interna
- Problemas de autenticação

---

# PX-Backup

## Instalação falha

### Verificações

```bash
oc get pods -n px-backup
```

### Possíveis Causas

- CPU sem AVX
- Recursos insuficientes
- Storage indisponível

---

## Backup falha

### Verificações

- Backup Location
- Object Storage
- Credenciais

### Possíveis Causas

- Bucket inacessível
- Credenciais inválidas
- Permissões insuficientes

---

## Restore falha

### Possíveis Causas

- Namespace destino
- Recursos insuficientes
- Conflito de objetos

---

# Comandos Rápidos

## OpenShift

```bash
oc get nodes

oc get co

oc get mcp

oc get events -A --sort-by='.lastTimestamp'
```

---

## Portworx

```bash
pxctl status

pxctl cluster list

pxctl service pool show
```

---

## Storage

```bash
oc get pvc -A

oc get pv

oc get sc
```

---

## MetalLB

```bash
oc get metallb -A

oc get ipaddresspool -A

oc get l2advertisement -A
```

---

## Virtualization

```bash
oc get vm -A

oc get vmi -A

oc get dv -A
```

---

# Histórico de Problemas do Lab POA

## Data

### Problema

### Sintoma

### Causa

### Solução

### Responsável

### Observações

---

## Data

### Problema

### Sintoma

### Causa

### Solução

### Responsável

### Observações