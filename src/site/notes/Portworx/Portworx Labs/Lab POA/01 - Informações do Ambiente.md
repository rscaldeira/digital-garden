---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/01 - Informações do Ambiente/","dg-note-properties":{}}
---


# **Informações do Ambiente**

## **Objetivo**

Este documento centraliza as informações de referência do ambiente Lab POA.

Seu objetivo é fornecer uma visão rápida da infraestrutura, versões, componentes, endereçamento e decisões arquiteturais adotadas para o laboratório.

## **Documentos Relacionados**

- [[Portworx/Portworx Labs/Lab POA/00 - Visão Geral\|00 - Visão Geral]]
- [[Portworx/Portworx Labs/Lab POA/00 - Build Order\|00 - Build Order]]
- [[Portworx/Portworx Labs/Lab POA/02 - Arquitetura Alvo\|02 - Arquitetura Alvo]]
- [[Portworx/Portworx Labs/Lab POA/13 - Day2 Operations\|13 - Day2 Operations]]

---

# **Localização**

Local:

- Escritório Pure Storage Porto Alegre

Responsáveis locais:

- Gabriel
- Manoel

Apoio técnico:

- Rodrigo Caldeira

---

# **Hardware**

## **Cluster OpenShift**

Topologia:

- Cluster compacto (Compact Cluster)

Quantidade de nós:

- 3 nós

Funções:

- Control Plane
- Worker
- Infra

Todos os nós executam simultaneamente:

- Control Plane
- Worker
- Infra

---

# **Rede e Endereçamento**

## **OpenShift**

Cluster Name:

-   
    
    ---
    

Base Domain:

-   
    
    ---
    

API VIP:

-   
    
    ---
    

Ingress VIP:

-   
    
    ---
    

Machine Network:

-   
    
    ---
    

Cluster Network:

-   
    
    ---
    

Service Network:

-   
    
    ---
    

Gateway:

-   
    
    ---
    

DNS Server:

-   
    
    ---
    

VLAN:

-   
    
    ---
    

---

## **Nodes**

|**Hostname**|**IP**|**Função**|
|---|---|---|
|master01||Control Plane / Worker|
|master02||Control Plane / Worker|
|master03||Control Plane / Worker|

---

## **MetalLB**

Pool Name:

-   
    
    ---
    

Range:

-   
    
    ---
    

L2Advertisement:

-   
    
    ---
    

---

## **FlashArray**

Management IP:

-   
    
    ---
    

Protocol:

- iSCSI / FC / NVMe/TCP / NVMe/RDMA

---

# **OpenShift**

Versão:

- OpenShift Container Platform 4.21.15

Método de instalação:

- Assisted Installer

Topologia:

- Compact Cluster (3 Nodes)

Networking:

- OVNKubernetes

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/checklist_instalacao_openshift_4_21_assisted_installer\|arquivos/openshift/checklist_instalacao_openshift_4_21_assisted_installer]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/runbook_openshift_4_21_assisted_installer_compacto\|arquivos/openshift/runbook_openshift_4_21_assisted_installer_compacto]]

---

# **Storage**

## **Backend Principal**

Array:

- Pure Storage FlashArray X20

Integração:

- FlashArray Cloud Drives (FACD)

Protocolos suportados:

- iSCSI
- Fibre Channel (FC)
- NVMe/TCP
- NVMe/RDMA

Protocolo utilizado:

- A definir durante a implantação

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray Configuration\|arquivos/flasharray/FlashArray Configuration]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - pure.json\|arquivos/flasharray/FlashArray - pure.json]]
- [[arquivos/flasharray/FlashArray - Secret px-pure-secret\|arquivos/flasharray/FlashArray - Secret px-pure-secret]]

---

# **Portworx**

Versão:

- Portworx Enterprise 3.6

Arquitetura:

- PX-StoreV2
- Internal KVDB
- FlashArray Cloud Drives

Deployment:

- Portworx Operator
- StorageCluster gerado via Portworx Central Spec Generator

Secret obrigatório:

- px-pure-secret

MachineConfigs obrigatórias:

- Multipath
- Udev Rules

Monitoring:

- User Defined Monitoring habilitado

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray\|arquivos/portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/portworx/runbook_portworx_enterprise_3_6_openshift_4_21_15_flasharray\|arquivos/portworx/runbook_portworx_enterprise_3_6_openshift_4_21_15_flasharray]]

---

# **OpenShift Virtualization**

Versão:

- Compatível com OpenShift 4.21

Namespace:

- openshift-cnv

Objetivos do laboratório:

- Fedora VM
- Windows VM
- CDI Import
- Snapshot
- Clone
- Live Migration
- Node Drain Testing

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/checklist_instalacao_openshift_virtualization_4_21\|arquivos/virtualization/checklist_instalacao_openshift_virtualization_4_21]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/runbook_openshift_virtualization_4_21\|arquivos/virtualization/runbook_openshift_virtualization_4_21]]

---

# **MetalLB**

Método de instalação:

- MetalLB Operator

Modo:

- Layer2

Objetivo:

- Disponibilizar Services do tipo LoadBalancer em ambiente bare metal

Componentes:

- MetalLB
- IPAddressPool
- L2Advertisement

Consumidores previstos:

- PX-Central
- PX-Backup
- Aplicações de demonstração
- Workloads futuros

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/checklist_instalacao_metallb_openshift_4_21_15\|arquivos/metallb/checklist_instalacao_metallb_openshift_4_21_15]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/runbook_metallb_openshift_4_21_15\|arquivos/metallb/runbook_metallb_openshift_4_21_15]]

---

# **PX-Central**

Objetivo:

- Administração centralizada do ecossistema Portworx

Publicação:

- Service LoadBalancer via MetalLB

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-central/checklist_instalacao_px_central\|arquivos/px-central/checklist_instalacao_px_central]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-central/runbook_px_central_openshift_metallb\|arquivos/px-central/runbook_px_central_openshift_metallb]]

---

# **PX-Backup**

Versão:

- PX-Backup 3.0

Objetivo:

- Backup e Restore de workloads Kubernetes
- Testes de proteção de dados

Publicação:

- Service LoadBalancer via MetalLB

Pré-requisito importante:

- CPUs devem possuir suporte AVX

Validação:

```bash
lscpu | grep -i avx
```

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/checklist_instalacao_px_backup\|arquivos/px-backup/checklist_instalacao_px_backup]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/runbook_px_backup_openshift_metallb\|arquivos/px-backup/runbook_px_backup_openshift_metallb]]

---

# **Storage Classes Planejadas**

## **Aplicações**

- px-app-repl1
- px-app-repl2
- px-app-repl3

## **Virtualização**

- px-vm-block-repl1
- px-vm-block-repl2
- px-rwx-file-kubevirt
- px-cdi-scratch

Documentação:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/Storage Classes Overview\|arquivos/storageclasses/Storage Classes Overview]]

---

# **Testes Planejados**

## **OpenShift**

- Instalação
- Rebuild completo

## **MetalLB**

- LoadBalancer
- IP Allocation

## **Portworx**

- PVC
- Snapshot
- Clone
- FIO
- Replicação

## **OpenShift Virtualization**

- Fedora VM
- Windows VM
- Live Migration
- Node Drain
- Snapshot
- Clone
- CDI Import

## **PX-Backup**

- Instalação
- Registro de Cluster
- Backup
- Restore

Documentação:

- [[arquivos/tests\|arquivos/tests]]

---

# **Conectividade Externa**

Necessária para:

- Red Hat
- Portworx
- Download de imagens
- Licenciamento
- Telemetria

URLs importantes:

- registry.redhat.io
- quay.io
- install.portworx.com
- mirrors.portworx.com

---

# **Status Atual**

Fase atual:

- Documentação concluída
- Ambiente pronto para implantação
- Runbooks e checklists finalizados
- Build Order definido
- Day2 Operations definido

Próximo passo:

- Executar instalação do ambiente
- Validar matriz de testes
- Consolidar procedimento de rebuild completo do laboratório