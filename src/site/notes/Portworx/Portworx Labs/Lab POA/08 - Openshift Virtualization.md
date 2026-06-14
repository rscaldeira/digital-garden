---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization/","dg-note-properties":{}}
---



# OpenShift Virtualization

## Objetivo

Implantar OpenShift Virtualization 4.21 no Lab POA para demonstrar a execução de máquinas virtuais sobre OpenShift utilizando Portworx como plataforma de armazenamento persistente.

O objetivo não é apenas criar VMs, mas demonstrar a convergência entre:

- Containers
- Máquinas Virtuais
- Storage definido por software
- Backup
- Snapshots
- Clones
- Mobilidade de workloads

na mesma plataforma.

---

# Casos de Uso

## Virtualização Moderna

Demonstrar OpenShift como plataforma única para:

- Containers
- Máquinas Virtuais
- Aplicações modernas
- Aplicações legadas

---

## VMware Replacement

Permitir demonstrações relacionadas a:

- VMware Migration
- OpenShift Virtualization
- Modernização de Datacenter
- Consolidação de Plataformas

---

## Portworx para Virtualização

Demonstrar:

- Persistência
- Replicação
- Snapshots
- Clones
- Alta disponibilidade
- Storage Mobility

---

## PX-Backup

Demonstrar:

- Backup de VMs
- Restore de VMs
- Proteção de workloads virtualizados

---

# Dependências

Antes de iniciar:

- [[Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift\|05 - Instalação OpenShift]]
- [[Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB\|06 - Instalação MetalLB]]
- [[Portworx/Portworx Labs/Lab POA/07 - Instalação Portworx\|07 - Instalação Portworx]]

Validações obrigatórias:

- Cluster OpenShift saudável
- Portworx Online
- StorageClasses criadas
- Snapshot Controller disponível
- MetalLB funcional

---

# Arquitetura Definida

## OpenShift Virtualization

Instalação:

- OperatorHub
- Namespace openshift-cnv
- Canal stable
- Approval Strategy Automatic

---

## Storage

Storage principal:

- Portworx

StorageClasses planejadas:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-vm-block-repl1.yaml\|Arquivos/storageclasses/px-vm-block-repl1.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-vm-block-repl2.yaml\|Arquivos/storageclasses/px-vm-block-repl2.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-rwx-file-kubevirt.yaml\|Arquivos/storageclasses/px-rwx-file-kubevirt.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-cdi-scratch.yaml\|Arquivos/storageclasses/px-cdi-scratch.yaml]]

---

## Templates

Artefatos do laboratório:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Fedora VM.yaml\|Arquivos/virtualization/Fedora VM.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Windows VM.yaml\|Arquivos/virtualization/Windows VM.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/CDI Import.yaml\|Arquivos/virtualization/CDI Import.yaml]]

---

# Documentação Oficial do Projeto

## Checklist

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/checklist_instalacao_openshift_virtualization_4_21\|Arquivos/virtualization/checklist_instalacao_openshift_virtualization_4_21]]

## Runbook

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/runbook_openshift_virtualization_4_21\|Arquivos/virtualization/runbook_openshift_virtualization_4_21]]

---

# Estratégia do Lab POA

O ambiente será utilizado para demonstrar:

## Nível 1

- Criação de VM
- Console Web
- Persistência Portworx

---

## Nível 2

- Snapshot
- Clone
- Import de imagens

---

## Nível 3

- Live Migration
- Drain de Node
- Alta Disponibilidade

---

## Nível 4

- Backup com PX-Backup
- Restore de VM
- Proteção de workloads

---

# Fluxo de Implantação

## Etapa 1

Instalar OpenShift Virtualization Operator.

Validação:

```bash
oc get csv -n openshift-cnv
```

Resultado esperado:

- CSV Succeeded

---

## Etapa 2

Validar HyperConverged.

```bash
oc get hco -n openshift-cnv
```

Resultado esperado:

- Available=True
- Degraded=False
- Progressing=False

---

## Etapa 3

Validar CDI.

```bash
oc get cdi
oc get pods -n openshift-cnv
```

Resultado esperado:

- CDI Available
- UploadProxy Running

---

## Etapa 4

Validar StorageClasses.

```bash
oc get sc
```

Resultado esperado:

- px-vm-block-repl1
- px-vm-block-repl2
- px-rwx-file-kubevirt
- px-cdi-scratch

---

## Etapa 5

Importar imagem Fedora.

Arquivo:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/CDI Import.yaml\|Arquivos/virtualization/CDI Import.yaml]]

---

## Etapa 6

Criar VM Fedora.

Arquivo:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Fedora VM.yaml\|Arquivos/virtualization/Fedora VM.yaml]]

---

## Etapa 7

Criar VM Windows.

Arquivo:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Windows VM.yaml\|Arquivos/virtualization/Windows VM.yaml]]

---

# Validações Obrigatórias

## VM Creation

Validar:

- Boot
- Console
- Rede
- Storage

---

## Storage

Validar:

- PVC criada
- PV criada
- Disco provisionado via Portworx

---

## Snapshot

Executar snapshot.

Validar:

- Snapshot criada
- Snapshot restaurável

---

## Clone

Executar clone.

Validar:

- Clone funcional
- Clone inicializável

---

## Live Migration

Executar migração entre nós.

Validar:

- Migração concluída
- VM acessível após migração
- Comportamento observado documentado

Observação:

O comportamento de Live Migration depende da combinação entre OpenShift Virtualization, KubeVirt e Portworx. Validar no laboratório o comportamento efetivo da StorageClass utilizada antes de assumir suporte a migração sem interrupção.

---

## Drain de Node

Executar:

```bash
oc adm drain <node> --ignore-daemonsets --delete-emptydir-data --force
```

Validar:

- Comportamento das VMs
- Migração automática
- Recuperação do cluster

Retorno:

```bash
oc adm uncordon <node>
```

---

# Entregáveis da Etapa

Ao final registrar:

## Operator

- Versão instalada
- Canal utilizado

## Virtualização

- HyperConverged saudável
- CDI saudável

## Storage

- StorageClasses utilizadas
- SnapshotClass utilizada

## VMs

- Fedora VM criada
- Windows VM criada

## Testes

- Snapshot validado
- Clone validado
- Live Migration validada
- Drain validado

---

# Resultado Esperado

O ambiente deve estar pronto para:

- [[Portworx/Portworx Labs/Lab POA/09 - PX-Central\|09 - PX-Central]]
- [[Portworx/Portworx Labs/Lab POA/10 - PX-Backup\|10 - PX-Backup]]

e para demonstrações de:

- VMware Replacement
- OpenShift Virtualization
- Portworx for VMs
- Snapshot
- Clone
- Alta Disponibilidade
- Backup de Máquinas Virtuais

---

# Status

Status atual:

- ☐ Não iniciado
- ☐ Em andamento
- ☐ Concluído

Data da instalação:

Responsável:

Observações: