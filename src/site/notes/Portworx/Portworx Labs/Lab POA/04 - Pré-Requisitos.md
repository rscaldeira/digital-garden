---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/04 - Pré-Requisitos/","dg-note-properties":{}}
---



# **Pré-Requisitos**

## **Objetivo**

Este documento consolida os pré-requisitos mínimos necessários para iniciar a implantação do Lab POA.

Os detalhes de instalação de cada componente estão documentados nos respectivos checklists e runbooks na pasta `Arquivos`.

---

# **Documentos Relacionados**

## **OpenShift**

- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/checklist_instalacao_openshift_4_21_assisted_installer\|Arquivos/OpenShift/checklist_instalacao_openshift_4_21_assisted_installer]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/runbook_openshift_4_21_assisted_installer_compacto\|Arquivos/OpenShift/runbook_openshift_4_21_assisted_installer_compacto]]

## **MetalLB**

- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/checklist_instalacao_metallb_openshift_4_21_15\|Arquivos/MetalLB/checklist_instalacao_metallb_openshift_4_21_15]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/runbook_metallb_openshift_4_21_15\|Arquivos/MetalLB/runbook_metallb_openshift_4_21_15]]

## **Portworx**

- [[Portworx/Portworx Labs/Lab POA/Arquivos/portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray\|Arquivos/Portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/portworx/runbook_portworx_enterprise_3_6_openshift_4_21_15_flasharray\|Arquivos/Portworx/runbook_portworx_enterprise_3_6_openshift_4_21_15_flasharray]]

## **OpenShift Virtualization**

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/checklist_instalacao_openshift_virtualization_4_21\|Arquivos/Virtualization/checklist_instalacao_openshift_virtualization_4_21]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/runbook_openshift_virtualization_4_21\|Arquivos/Virtualization/runbook_openshift_virtualization_4_21]]

## **PX-Backup**

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/checklist_instalacao_px_backup\|Arquivos/PX-Backup/checklist_instalacao_px_backup]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/runbook_px_backup_openshift_metallb\|Arquivos/PX-Backup/runbook_px_backup_openshift_metallb]]

---

# **Hardware**

## **Cluster**

Pré-requisitos mínimos:

- 3 servidores disponíveis
- Hardware validado
- Firmware atualizado quando aplicável
- Virtualização habilitada na BIOS (Intel VT-x / AMD-V)
- CPUs compatíveis com OpenShift Virtualization
- CPUs compatíveis com AVX (PX-Backup)

Pendências:

- Confirmar modelo dos servidores
- Confirmar CPUs
- Confirmar memória RAM
- Confirmar interfaces de rede

Referência:

- [[Portworx/Portworx Labs/Lab POA/01 - Informações do Ambiente\|01 - Informações do Ambiente]]
- [[Portworx/Portworx Labs/Lab POA/03 - Informações Pendentes\|03 - Informações Pendentes]]

---

# **Rede**

Pré-requisitos:

- Conectividade entre os três nós
- Gateway funcional
- DNS funcional
- DHCP funcional
- Conectividade com Internet
- Resolução DNS externa funcional

Pendências:

- Definição final da VLAN
- Definição do range MetalLB
- Definição dos VIPs do OpenShift

Referência:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/network/Network Planning\|Arquivos/network/Network Planning]]
- [[Portworx/Portworx Labs/Lab POA/03 - Informações Pendentes\|03 - Informações Pendentes]]

---

# **OpenShift**

Pré-requisitos:

- Conta Red Hat válida
- Pull Secret válido
- Acesso ao Assisted Installer
- Acesso ao Red Hat Hybrid Cloud Console
- OpenShift 4.21.15 disponível

Referência:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/checklist_instalacao_openshift_4_21_assisted_installer\|Arquivos/OpenShift/checklist_instalacao_openshift_4_21_assisted_installer]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/runbook_openshift_4_21_assisted_installer_compacto\|Arquivos/OpenShift/runbook_openshift_4_21_assisted_installer_compacto]]

---

# **FlashArray**

Pré-requisitos:

- FlashArray acessível
- Management IP acessível
- Conectividade validada
- Credenciais administrativas disponíveis
- API Token disponível

Pendências:

- Definição final do protocolo:
    - FC
    - iSCSI
    - NVMe/TCP
    - NVMe/RDMA

Referência:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray Configuration\|Arquivos/FlashArray/FlashArray Configuration]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - pure.json\|Arquivos/FlashArray/FlashArray - pure.json]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - Secret px-pure-secret.yaml\|FlashArray - Secret px-pure-secret.yaml]]

---

# **Portworx**

Pré-requisitos:

- MachineConfig Multipath pronta
- MachineConfig Udev pronta
- cluster-monitoring-config disponível
- PX-StoreV2 definido
- FlashArray Cloud Drives definidos
- Pure Secret preparado

Referência:

- [[Arquivos/OpenShift/machineconfig\|Arquivos/OpenShift/machineconfig]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/cluster-monitoring-config.yaml\|cluster-monitoring-config.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray\|Arquivos/Portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray]]

---

# **OpenShift Virtualization**

Pré-requisitos:

- Portworx saudável
- StorageCluster Online
- Snapshot Controller instalado
- StorageClasses disponíveis:
    - px-vm-block-repl1
    - px-vm-block-repl2
    - px-rwx-file-kubevirt
    - px-cdi-scratch

Referência:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/checklist_instalacao_openshift_virtualization_4_21\|Arquivos/Virtualization/checklist_instalacao_openshift_virtualization_4_21]]

---

# **PX-Backup**

Pré-requisitos:

- CPUs com suporte AVX
- StorageClass disponível
- Namespace definido
- MetalLB funcional

Referência:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/checklist_instalacao_px_backup\|Arquivos/PX-Backup/checklist_instalacao_px_backup]]

---

# **Ferramentas**

Estação administrativa:

- oc
- kubectl
- jq
- yq
- virtctl
- navegador web

Validações sugeridas:

```bash
oc version
kubectl version --client
jq --version
yq --version
virtctl version
```

---

# **Acessos Necessários**

Disponíveis antes do início da implantação:

- Console dos servidores
- iDRAC / iLO / BMC
- DNS
- DHCP
- FlashArray
- Red Hat Hybrid Cloud Console
- Assisted Installer
- OpenShift Console (após instalação)

---

# **NTP**

Validar sincronismo utilizando:

- time1.purestorage.com
- time2.purestorage.com
- time3.purestorage.com

Validação sugerida:

```bash
chronyc sources
chronyc tracking
```

---

# **Critério para Início da Implantação**

A implantação pode começar quando:

- Hardware estiver disponível
- Rede estiver funcional
- DNS estiver funcional
- DHCP estiver funcional
- FlashArray estiver acessível
- Pull Secret estiver disponível
- Credenciais administrativas estiverem disponíveis
- Estação administrativa estiver preparada
- Pré-requisitos do checklist do OpenShift estiverem atendidos

Referências:

- [[Portworx/Portworx Labs/Lab POA/00 - Visão Geral\|00 - Visão Geral]]
- [[Portworx/Portworx Labs/Lab POA/01 - Informações do Ambiente\|01 - Informações do Ambiente]]
- [[Portworx/Portworx Labs/Lab POA/02 - Arquitetura Alvo\|02 - Arquitetura Alvo]]
- [[Portworx/Portworx Labs/Lab POA/03 - Informações Pendentes\|03 - Informações Pendentes]]