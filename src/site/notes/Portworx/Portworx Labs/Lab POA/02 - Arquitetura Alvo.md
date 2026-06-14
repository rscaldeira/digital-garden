---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/02 - Arquitetura Alvo/","dg-note-properties":{}}
---




# **Arquitetura Alvo**

## **Objetivo**

Criar um ambiente permanente de demonstração, validação e treinamento Pure Storage no escritório de Porto Alegre.

O ambiente será utilizado para:

- Demonstrações para clientes
- Testes internos
- Validação de funcionalidades
- Treinamentos
- Hands-on
- Provas de conceito
- Reproduções de cenários de clientes

Como a licença do OpenShift possui validade limitada, o ambiente poderá ser reconstruído periodicamente. Portanto, a arquitetura deve priorizar:

- Simplicidade operacional
- Facilidade de reconstrução
- Baixa dependência de configurações manuais
- Documentação completa do ambiente

---

# **Plataforma**

## **OpenShift**

Versão:

- OpenShift Container Platform 4.21.15

Método de instalação:

- Assisted Installer

Networking:

- OVNKubernetes

Topologia:

- Compact Cluster

---

# **Cluster**

## **Estrutura**

Quantidade de nós:

- 3

Modelo:

- Compact Cluster

Funções por nó:

- Control Plane
- Worker
- Infra

Todos os serviços executam nos mesmos servidores.

---

# **Arquitetura de Alto Nível**

```text
+------------------------------------------------+
|                OpenShift 4.21.15               |
|                                                |
|  +------------------------------------------+  |
|  | OpenShift Virtualization                 |  |
|  | Fedora / Windows VMs                     |  |
|  +------------------------------------------+  |
|                                                |
|  +------------------------------------------+  |
|  | PX-Backup                                |  |
|  +------------------------------------------+  |
|                                                |
|  +------------------------------------------+  |
|  | PX-Central                               |  |
|  +------------------------------------------+  |
|                                                |
|  +------------------------------------------+  |
|  | Portworx Enterprise 3.6                  |  |
|  | PX-StoreV2                               |  |
|  +------------------------------------------+  |
|                                                |
|  +------------------------------------------+  |
|  | MetalLB                                  |  |
|  +------------------------------------------+  |
+------------------------------------------------+
                     |
                     |
                     v
            FlashArray X20
```

---

# **Storage**

## **Backend Principal**

Array:

- Pure Storage FlashArray X20

Modelo de integração:

- FlashArray Cloud Drives (FACD)

Backend Portworx:

- PX-StoreV2

KVDB:

- Internal KVDB

Metadata:

- Definida durante a geração da spec do StorageCluster
- Pode utilizar SSD local dedicado ou metadata provisionada pelo FlashArray, conforme o desenho final

Protocolos suportados:

- iSCSI
- Fibre Channel
- NVMe/TCP
- NVMe/RDMA

Protocolo utilizado:

- A definir durante a implantação

---

# **Portworx Enterprise**

Versão:

- Portworx Enterprise 3.6

Método de instalação:

- Portworx Operator

Configuração:

- StorageCluster gerado via Portworx Central Spec Generator

Dependências:

- MachineConfig Multipath
- MachineConfig Udev Rules
- Pure Secret (px-pure-secret)
- FlashArray acessível pelos nós

Objetivos:

- Persistent Volumes
- Snapshots
- Clones
- Storage para VMs
- Integração com PX-Backup

---

# **MetalLB**

Método de instalação:

- MetalLB Operator

Modo:

- Layer2

Objetivos:

- Disponibilizar Services LoadBalancer em ambiente bare metal

Consumidores:

- PX-Central
- PX-Backup
- Aplicações de demonstração
- Serviços futuros do laboratório

Componentes:

- MetalLB
- IPAddressPool
- L2Advertisement

---

# **OpenShift Virtualization**

Método:

- OpenShift Virtualization Operator

Namespace:

- openshift-cnv

Objetivos:

- Fedora VM
- Windows VM
- CDI Import
- Snapshots
- Clones
- Live Migration
- Node Drain Validation

Storage principal:

- px-vm-block-repl2

Storage para laboratório:

- px-vm-block-repl1

Storage RWX:

- px-rwx-file-kubevirt

CDI:

- px-cdi-scratch

---

# **PX-Central**

Objetivo:

- Gerenciamento centralizado do ambiente Portworx

Publicação:

- Service LoadBalancer via MetalLB

Funções previstas:

- Monitoramento
- Administração do cluster
- Visualização de volumes
- Health Checks

---

# **PX-Backup**

Versão:

- PX-Backup 3.0

Objetivo:

- Backup e Restore de workloads Kubernetes
- Backup de aplicações persistentes
- Validação de integrações Portworx

Publicação:

- Service LoadBalancer via MetalLB

Pré-requisito:

- CPUs com suporte AVX

---

# **Storage Classes**

## **Aplicações**

- px-app-repl1
- px-app-repl2
- px-app-repl3

## **Virtualização**

- px-vm-block-repl1
- px-vm-block-repl2
- px-rwx-file-kubevirt
- px-cdi-scratch

---

# **Rede**

Modelo:

- Ambiente isolado de laboratório

Load Balancer:

- MetalLB Layer2

VIPs OpenShift:

- API VIP
- Ingress VIP

DNS:

- Publicação conforme necessidade operacional

---

# **NTP**

Servidores planejados:

- time1.purestorage.com
- time2.purestorage.com
- time3.purestorage.com

---

# **Internet**

Necessária para:

- Instalação do OpenShift
- Download de operadores
- Atualizações
- Instalação de componentes adicionais
- Acesso ao Portworx Central
- Download de imagens
- Licenciamento
- Telemetria

URLs relevantes:

- install.portworx.com
- mirrors.portworx.com
- registry.redhat.io
- quay.io

---

# **Objetivos de Validação**

O ambiente deverá permitir validar:

## **OpenShift**

- Instalação
- Rebuild completo

## **MetalLB**

- Services LoadBalancer
- Alocação de IPs

## **Portworx**

- PVCs
- Snapshots
- Clones
- PX-StoreV2
- FlashArray Cloud Drives

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
- Registro de cluster
- Backup
- Restore