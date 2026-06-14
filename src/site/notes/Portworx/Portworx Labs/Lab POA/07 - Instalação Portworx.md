---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/07 - Instalação Portworx/","dg-note-properties":{}}
---



# Instalação Portworx

## Objetivo

Implantar o Portworx Enterprise 3.6 sobre OpenShift 4.21.15 utilizando Pure Storage FlashArray como backend de armazenamento principal do Lab POA.

O objetivo é disponibilizar armazenamento persistente para:

- OpenShift Virtualization
- PX-Central
- PX-Backup
- Aplicações Kubernetes
- Testes de snapshots
- Testes de clones
- Testes de alta disponibilidade

Toda a instalação deve ser reproduzível através dos runbooks e checklists armazenados na pasta Arquivos.

---

# Arquitetura Definida

## Cluster OpenShift

- 3 Nodes
- Control Plane + Worker no mesmo host
- OpenShift 4.21.15
- Cluster compacto

## Portworx

- Portworx Enterprise 3.6
- PX-StoreV2
- Internal KVDB
- FlashArray Cloud Drives
- Operator Installation
- StorageCluster gerado via Portworx Central Spec Generator

## FlashArray

Backend principal:

- Pure FlashArray

Protocolo:

- iSCSI

> Ajustar caso FC ou NVMe/TCP sejam utilizados.

---

# Dependências

Antes de iniciar:

- [[Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift\|05 - Instalação OpenShift]]
- [[Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB\|06 - Instalação MetalLB]]

Validação:

```bash
oc get nodes
oc get co
oc get mcp
```

Resultado esperado:

- Nodes Ready
- Cluster Operators Healthy
- MCP Updated=True
- MCP Degraded=False

---

# Planejamento de Storage

## FlashArray

Validar:

- Management Endpoint
- API Token
- Conectividade dos nodes

Documentos relacionados:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray Configuration\|Arquivos/flasharray/FlashArray Configuration]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - pure.json\|Arquivos/flasharray/FlashArray - pure.json]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - Secret px-pure-secret.yaml\|FlashArray - Secret px-pure-secret.yaml]]

---

## PX-StoreV2

Modelo adotado:

- Cloud Drives no FlashArray
- Metadata Device dedicado

Validar:

- SSD local disponível para metadata
- Device visível nos hosts
- Device em modo RAW

Validação:

```bash
lsblk
```

---

# MachineConfigs Obrigatórias

Antes da instalação do Portworx aplicar:

- Multipath
- Udev Rules

Localização:

- [[Arquivos/openshift/machineconfig\|Arquivos/openshift/machineconfig]]

Após aplicação:

```bash
oc get mcp
```

Resultado esperado:

- Updated=True
- Updating=False
- Degraded=False

---

# Monitoramento OpenShift

Antes do Operator:

Aplicar:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/cluster-monitoring-config.yaml\|Arquivos/openshift/cluster-monitoring-config.yaml]]

Objetivo:

Habilitar User Workload Monitoring necessário para o Portworx.

Validação:

```bash
oc get pods -n openshift-user-workload-monitoring
```

---

# Documentação Oficial do Projeto

## Checklist

- [[Portworx/Portworx Labs/Lab POA/Arquivos/portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray\|Arquivos/portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray]]

## Runbook

- [[Portworx/Portworx Labs/Lab POA/Arquivos/portworx/runbook_portworx_enterprise_3_6_openshift_4_21_15_flasharray\|Arquivos/portworx/runbook_portworx_enterprise_3_6_openshift_4_21_15_flasharray]]

## Spec Generator

- [[Arquivos/portworx/spec-generator\|Arquivos/portworx/spec-generator]]

---

# Storage Classes Planejadas

Após instalação do Portworx:

## Aplicações

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl1.yaml\|Arquivos/storageclasses/px-app-repl1.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl2.yaml\|Arquivos/storageclasses/px-app-repl2.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl3.yaml\|Arquivos/storageclasses/px-app-repl3.yaml]]

## Virtualização

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-vm-block-repl1.yaml\|Arquivos/storageclasses/px-vm-block-repl1.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-vm-block-repl2.yaml\|Arquivos/storageclasses/px-vm-block-repl2.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-rwx-file-kubevirt.yaml\|Arquivos/storageclasses/px-rwx-file-kubevirt.yaml]]

## CDI

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-cdi-scratch.yaml\|Arquivos/storageclasses/px-cdi-scratch.yaml]]

## Referência

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/Storage Classes Overview\|Arquivos/storageclasses/Storage Classes Overview]]

---

# Validações Obrigatórias

## Saúde do Cluster

```bash
pxctl status
```

Resultado esperado:

- Cluster Online

---

## Nós

```bash
pxctl cluster list
```

Resultado esperado:

- Todos os nós Online

---

## Pools

```bash
pxctl service pool show
```

Resultado esperado:

- Pools Online

---

## StorageCluster

```bash
oc get storagecluster -A
```

Resultado esperado:

- StorageCluster Healthy

---

# Testes Funcionais

Executar os testes armazenados em:

## PVC Básico

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/PVC - Basic Test.yaml\|Arquivos/tests/PVC - Basic Test.yaml]]

## PVC RWX

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/PVC - RWX Test.yaml\|Arquivos/tests/PVC - RWX Test.yaml]]

## Snapshot

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/Snapshot Test.yaml\|Arquivos/tests/Snapshot Test.yaml]]

## Clone

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/Clone Test.yaml\|Arquivos/tests/Clone Test.yaml]]

## FIO

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/FIO Test.yaml\|Arquivos/tests/FIO Test.yaml]]

---

# Entregáveis da Etapa

Ao final registrar:

## Cluster

- Versão Portworx
- Versão OpenShift
- Versão FlashArray

## Storage

- Pool Size
- Metadata Device
- Tipo de acesso ao FlashArray

## Operação

- StorageClasses criadas
- PVC validada
- Snapshot validado
- Clone validado

---

# Resultado Esperado

O ambiente deve estar pronto para:

- [[Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization\|08 - OpenShift Virtualization]]
- [[Portworx/Portworx Labs/Lab POA/09 - PX-Central\|09 - PX-Central]]
- [[Portworx/Portworx Labs/Lab POA/10 - PX-Backup\|10 - PX-Backup]]

Com:

- Storage persistente funcional
- Snapshots funcionais
- Clones funcionais
- Replicação funcional
- Integração FlashArray operacional

---

# Status

Status atual:

- ☐ Não iniciado
- ☐ Em andamento
- ☐ Concluído

Data da instalação:

Responsável:

Observações: