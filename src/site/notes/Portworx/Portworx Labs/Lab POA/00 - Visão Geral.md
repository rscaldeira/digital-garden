---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/00 - Visão Geral/","dg-note-properties":{}}
---



# **Lab POA**

## **Objetivo**

Criar e manter um laboratório permanente de OpenShift, Portworx e OpenShift Virtualization no escritório da Pure Storage em Porto Alegre.

O ambiente será utilizado para:

- Demonstrações técnicas
- Validação de funcionalidades
- Testes de integração
- Treinamentos internos
- Reproduções de cenários de clientes
- Estudos de arquitetura
- Hands-on de OpenShift Virtualization e Portworx

Como as licenças de OpenShift podem variar ao longo do tempo, o objetivo do projeto não é apenas manter um ambiente funcional, mas também documentar todo o processo de implantação para permitir reconstruções rápidas e reproduzíveis sempre que necessário.

---

## **Responsáveis**

- Gabriel – responsável local pelo laboratório
- Manoel – responsável local pelo laboratório
- Rodrigo – arquitetura, documentação e apoio técnico remoto/presencial

---

## **Arquitetura alvo**

### **Plataforma**

- OpenShift Container Platform 4.21.15
- MetalLB
- Portworx Enterprise 3.6
- PX-StoreV2
- Pure Storage FlashArray
- OpenShift Virtualization
- PX-Backup 3.0

### **Storage**

Backend principal:

- Pure Storage FlashArray

Provisionamento:

- FlashArray Cloud Drives (FACD)

Classes de storage padrão:

- px-app-repl1
- px-app-repl2
- px-app-repl3
- px-vm-block-repl1
- px-vm-block-repl2
- px-rwx-file-kubevirt
- px-cdi-scratch

### **Virtualização**

O OpenShift Virtualization será utilizado para:

- VMs Linux
- VMs Windows
- Testes de Live Migration
- Snapshots
- Clones
- CDI Imports

### **Backup**

PX-Backup será utilizado para:

- Proteção de workloads Kubernetes
- Testes de backup e restore
- Validação de integrações Portworx

---

## **Objetivos de validação**

O laboratório deve permitir validar:

### **OpenShift**

- Instalação via Assisted Installer
- Rebuild completo do ambiente

### **MetalLB**

- Serviços LoadBalancer
- Publicação de aplicações

### **Portworx**

- PX-StoreV2
- FlashArray Cloud Drives
- StorageClasses
- PVCs
- Snapshots
- Clones
- Replicação

### **OpenShift Virtualization**

- Fedora VM
- Windows VM
- CDI Import
- Live Migration
- Node Drain
- Snapshots
- Clones

### **PX-Backup**

- Instalação
- Registro de clusters
- Backup
- Restore

---

## **Estratégia operacional**

### **Primeira implantação**

1. Instalar OpenShift
2. Configurar MetalLB
3. Preparar integração FlashArray
4. Instalar Portworx
5. Criar StorageClasses
6. Instalar OpenShift Virtualization
7. Validar VMs
8. Instalar PX-Central
9. Instalar PX-Backup
10. Executar testes funcionais

### **Operação contínua**

Após a implantação inicial, todo o conhecimento operacional deverá estar documentado nos runbooks, checklists e templates armazenados neste vault.

O objetivo é permitir que qualquer membro do time consiga reconstruir o ambiente seguindo apenas a documentação.

---

## **Estrutura da documentação**

A pasta `Arquivos` contém:

- Runbooks de instalação
- Checklists de validação
- Templates YAML
- Configurações de StorageClasses
- Configurações de FlashArray
- Artefatos de Virtualização
- Testes operacionais

Toda alteração relevante no ambiente deve ser refletida na documentação correspondente.

---

## **Status do projeto**

Objetivo atual:

- Finalizar documentação
- Executar implantação completa
- Validar matriz de testes
- Consolidar procedimento de rebuild do laboratório