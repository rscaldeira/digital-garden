---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/10 - PX-Backup/","dg-note-properties":{}}
---


# PX-Backup

## Objetivo

Implantar o PX-Backup 3.0 como plataforma de proteção de dados para workloads Kubernetes e máquinas virtuais do Lab POA.

O objetivo é demonstrar:

- Backup de aplicações Kubernetes
- Backup de volumes Portworx
- Backup de máquinas virtuais
- Restore de aplicações
- Restore de VMs
- Proteção operacional de workloads
- Integração com Portworx Enterprise

O PX-Backup representa a camada de Data Protection da arquitetura do laboratório.

---

# Dependências

Antes de iniciar:

- [[Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift\|05 - Instalação OpenShift]]
- [[Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB\|06 - Instalação MetalLB]]
- [[Portworx/Portworx Labs/Lab POA/07 - Instalação Portworx\|07 - Instalação Portworx]]
- [[Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization\|08 - OpenShift Virtualization]]
- [[Portworx/Portworx Labs/Lab POA/09 - PX-Central\|09 - PX-Central]]

Validações obrigatórias:

```bash
oc get nodes
oc get co
oc get storagecluster -A
```

Resultado esperado:

- Cluster saudável
- StorageCluster Online
- Todos os nós Online

---

# Arquitetura Definida

## Plataforma

- PX-Backup 3.0
- OpenShift 4.21.15
- Portworx Enterprise 3.6

---

## Publicação

Método:

- Service LoadBalancer

Provider:

- MetalLB

Objetivo:

- Facilitar acesso externo
- Simplificar demonstrações
- Evitar dependência de Routes

---

## Integrações

PX-Backup será integrado com:

- OpenShift
- Portworx Enterprise
- OpenShift Virtualization
- PX-Central

---

# Documentação Oficial do Projeto

## Checklist

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/checklist_instalacao_px_backup\|Arquivos/px-backup/checklist_instalacao_px_backup]]

## Runbook

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/runbook_px_backup_openshift_metallb\|Arquivos/px-backup/runbook_px_backup_openshift_metallb]]

---

# Fluxo de Implantação

## Etapa 1

Validar Portworx.

```bash
oc get storagecluster -A
```

Resultado esperado:

- StorageCluster Online

---

## Etapa 2

Validar PX-Central.

Confirmar:

- Dashboard acessível
- Cluster visível
- Sem alertas críticos

---

## Etapa 3

Validar MetalLB.

```bash
oc get metallb -A
oc get ipaddresspool -A
oc get l2advertisement -A
```

Resultado esperado:

- MetalLB saudável
- Pool criado
- L2Advertisement criado

---

## Etapa 4

Executar instalação do PX-Backup.

Seguir:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/runbook_px_backup_openshift_metallb\|Arquivos/px-backup/runbook_px_backup_openshift_metallb]]

---

## Etapa 5

Validar pods.

```bash
oc get pods -n px-backup
```

Resultado esperado:

- Pods Running
- Pods Completed quando aplicável

---

## Etapa 6

Validar Service LoadBalancer.

```bash
oc get svc -n px-backup
```

Resultado esperado:

- EXTERNAL-IP atribuído
- IP vindo do pool MetalLB

---

## Etapa 7

Publicar DNS.

Registrar:

```text
Hostname:
IP:
```

Exemplo:

```text
px-backup.lab.local
192.168.x.x
```

---

# Casos de Uso Planejados

## Kubernetes Applications

Validar:

- Backup
- Restore
- Namespace Restore

---

## Persistent Volumes

Validar:

- Backup de PVC
- Restore de PVC

---

## OpenShift Virtualization

Validar:

- Backup de VM Fedora
- Backup de VM Windows
- Restore de VM

---

## Snapshots

Validar:

- Snapshot local
- Restore via snapshot

---

# Workloads Utilizados

## Fedora VM

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Fedora VM.yaml\|Arquivos/virtualization/Fedora VM.yaml]]

---

## Windows VM

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Windows VM.yaml\|Arquivos/virtualization/Windows VM.yaml]]

---

## PVC de Teste

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/PVC - Basic Test.yaml\|Arquivos/tests/PVC - Basic Test.yaml]]

---

# Validações Funcionais

## Login

Validar:

- Console acessível
- Autenticação funcional

---

## Cluster Discovery

Validar:

- Cluster visível
- Recursos descobertos

---

## Backup Location

Validar:

- Backup Location criada
- Estado Healthy

---

## Backup

Executar:

- Backup manual

Validar:

- Backup concluído
- Sem erros

---

## Restore

Executar:

- Restore de workload

Validar:

- Restore concluído
- Aplicação funcional

---

## Backup de VM

Executar:

- Backup da Fedora VM
- Backup da Windows VM

Validar:

- Backup concluído
- Restore funcional

---

# Entregáveis da Etapa

Ao final registrar:

## Instalação

- Versão PX-Backup
- Namespace utilizado

## Publicação

- LoadBalancer IP
- Hostname

## Operação

- Console acessível
- Backup Location criada
- Primeiro backup executado
- Primeiro restore executado

---

# Resultado Esperado

O ambiente deve estar pronto para demonstrações de:

- Kubernetes Backup
- Kubernetes Restore
- VM Backup
- VM Restore
- Data Protection
- Disaster Recovery
- Proteção de workloads Portworx

---

# Critérios de Aceite

- PX-Backup instalado
- Pods saudáveis
- Service LoadBalancer funcional
- Console acessível
- Cluster integrado
- Backup Location criada
- Backup executado
- Restore executado
- Backup de VM validado
- Restore de VM validado

---

# Status

Status atual:

- ☐ Não iniciado
- ☐ Em andamento
- ☐ Concluído

Data da instalação:

Responsável:

Observações: