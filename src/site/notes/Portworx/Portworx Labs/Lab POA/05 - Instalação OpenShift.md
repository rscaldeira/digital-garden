---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift/","dg-note-properties":{}}
---


# **Instalação OpenShift**

## **Objetivo**

Esta etapa cobre a implantação da plataforma OpenShift Container Platform que servirá de base para todo o Lab POA.

Após a conclusão desta fase, o cluster deverá estar operacional e pronto para receber:

- MetalLB
- Portworx Enterprise
- OpenShift Virtualization
- PX-Central
- PX-Backup

---

# **Versão Planejada**

OpenShift Container Platform:

- 4.21.15

Método de instalação:

- Assisted Installer

Topologia:

- Compact Cluster

Quantidade de nós:

- 3

Networking:

- OVNKubernetes

---

# **Dependências**

Antes de iniciar a instalação, validar:

- Hardware disponível
- Rede funcional
- DHCP funcional
- DNS funcional
- Pull Secret disponível
- API VIP definido
- Ingress VIP definido
- Acesso ao Red Hat Hybrid Cloud Console

Referências:

- [[Portworx/Portworx Labs/Lab POA/03 - Informações Pendentes\|03 - Informações Pendentes]]
- [[Portworx/Portworx Labs/Lab POA/04 - Pré-Requisitos\|04 - Pré-Requisitos]]

---

# **Documentação Oficial do Projeto**

Checklist:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/checklist_instalacao_openshift_4_21_assisted_installer\|Arquivos/OpenShift/checklist_instalacao_openshift_4_21_assisted_installer]]

Runbook:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/openshift/runbook_openshift_4_21_assisted_installer_compacto\|Arquivos/OpenShift/runbook_openshift_4_21_assisted_installer_compacto]]

---

# **Resultado Esperado**

Ao final desta etapa o cluster deverá apresentar:

Validação:

```bash
oc get nodes
oc get co
oc get mcp
```

Critérios mínimos:

- Todos os nodes em Ready
- Todos os Cluster Operators Available
- Nenhum Cluster Operator Degraded
- Machine Config Pools saudáveis

---

# **Entregáveis da Etapa**

Após a instalação registrar:

## **Cluster**

- Nome do cluster
- Base Domain
- API VIP
- Ingress VIP

## **Nós**

- Hostnames
- Endereços IP
- MAC Addresses

## **Acesso**

- URL do Console
- URL da API
- Credenciais iniciais

---

# **Próxima Etapa**

Após a validação do OpenShift seguir para:

[[Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB\|06 - Instalação MetalLB]]

---

# **Status**

Status atual:

- ☐ Não iniciado
- ☐ Em andamento
- ☐ Concluído

Data da instalação:

Responsável:

Observações: