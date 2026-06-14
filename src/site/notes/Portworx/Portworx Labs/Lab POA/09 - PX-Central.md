---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/09 - PX-Central/","dg-note-properties":{}}
---


# PX-Central

## Objetivo

Implantar o PX-Central como plataforma central de gerenciamento e observabilidade do ambiente Portworx do Lab POA.

O PX-Central será utilizado para:

- Monitoramento do cluster Portworx
- Visualização de volumes
- Visualização de pools
- Visualização de nós
- Troubleshooting operacional
- Demonstrações para clientes
- Base para integração com PX-Backup

O PX-Central será considerado componente padrão do laboratório.

---

# Dependências

Antes de iniciar:

- [[Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift\|05 - Instalação OpenShift]]
- [[Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB\|06 - Instalação MetalLB]]
- [[Portworx/Portworx Labs/Lab POA/07 - Instalação Portworx\|07 - Instalação Portworx]]
- [[Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization\|08 - OpenShift Virtualization]]

Validações obrigatórias:

```bash
oc get nodes
oc get co
oc get storagecluster -A
```

Resultado esperado:

- Cluster OpenShift saudável
- StorageCluster Online
- Todos os nós Portworx Online

---

# Arquitetura Definida

## Plataforma

- PX-Central
- OpenShift 4.21.15
- Portworx Enterprise 3.6

---

## Publicação

Método:

- Service LoadBalancer

Provider:

- MetalLB

Objetivo:

- Evitar dependência de Routes
- Facilitar acesso externo
- Facilitar demonstrações

---

## Integrações

PX-Central será integrado com:

- Portworx Enterprise
- OpenShift
- PX-Backup

---

# Documentação Oficial do Projeto

## Checklist

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-central/checklist_instalacao_px_central\|Arquivos/px-central/checklist_instalacao_px_central]]

## Runbook

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-central/runbook_px_central_openshift_metallb\|Arquivos/px-central/runbook_px_central_openshift_metallb]]

---

# Fluxo de Implantação

## Etapa 1

Validar Portworx.

```bash
oc get storagecluster -A

PX_POD=$(oc get pod -n portworx -l name=portworx -o jsonpath='{.items[0].metadata.name}')

oc exec -n portworx -it ${PX_POD} -c portworx -- /opt/pwx/bin/pxctl status
```

Resultado esperado:

- Cluster Online
- Todos os nós Online

---

## Etapa 2

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

## Etapa 3

Executar instalação do PX-Central.

Seguir:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-central/runbook_px_central_openshift_metallb\|Arquivos/px-central/runbook_px_central_openshift_metallb]]

---

## Etapa 4

Validar pods.

```bash
oc get pods -A | grep px
```

Resultado esperado:

- Pods Running
- Pods Completed quando aplicável

---

## Etapa 5

Validar Service LoadBalancer.

```bash
oc get svc -A
```

Resultado esperado:

- EXTERNAL-IP atribuído
- IP vindo do pool MetalLB

---

## Etapa 6

Publicar DNS.

Registrar:

```text
Hostname:
IP:
```

Exemplo:

```text
px-central.lab.local
192.168.x.x
```

---

# Validações Funcionais

## Dashboard

Validar:

- Login
- Navegação
- Dashboard principal

---

## Cluster

Validar:

- Nós Portworx visíveis
- Status saudável

---

## Storage

Validar:

- Volumes visíveis
- Pools visíveis
- Capacidade visível

---

## Storage Classes

Validar visualização das classes:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl1.yaml\|Arquivos/storageclasses/px-app-repl1.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl2.yaml\|Arquivos/storageclasses/px-app-repl2.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-app-repl3.yaml\|Arquivos/storageclasses/px-app-repl3.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-vm-block-repl1.yaml\|Arquivos/storageclasses/px-vm-block-repl1.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-vm-block-repl2.yaml\|Arquivos/storageclasses/px-vm-block-repl2.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-rwx-file-kubevirt.yaml\|Arquivos/storageclasses/px-rwx-file-kubevirt.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/px-cdi-scratch.yaml\|Arquivos/storageclasses/px-cdi-scratch.yaml]]

---

## Alertas

Validar:

- Health Status
- Warnings
- Capacity

---

# Entregáveis da Etapa

Ao final registrar:

## Instalação

- Versão PX-Central
- Namespace utilizado

## Publicação

- LoadBalancer IP
- Hostname

## Operação

- Dashboard acessível
- Cluster visível
- Volumes visíveis
- Pools visíveis

---

# Resultado Esperado

O ambiente deve estar pronto para:

- [[Portworx/Portworx Labs/Lab POA/10 - PX-Backup\|10 - PX-Backup]]

E para demonstrações de:

- Monitoramento Portworx
- Troubleshooting
- Capacity Planning
- Administração do ambiente

---

# Critérios de Aceite

- PX-Central instalado
- Pods saudáveis
- Service LoadBalancer funcional
- Dashboard acessível
- Cluster Portworx visível
- Nodes visíveis
- Pools visíveis
- Volumes visíveis
- Storage Classes visíveis
- Sem alertas críticos

---

# Status

Status atual:

- ☐ Não iniciado
- ☐ Em andamento
- ☐ Concluído

Data da instalação:

Responsável:

Observações: