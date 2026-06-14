---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB/","dg-note-properties":{}}
---



# **Instalação MetalLB**

## **Objetivo**

Implementar o MetalLB como provedor de serviços `LoadBalancer` para o ambiente OpenShift Bare Metal do Lab POA.

O MetalLB será utilizado para disponibilizar IPs externos para componentes que necessitam exposição direta na rede, incluindo:

- PX-Central
- PX-Backup
- Aplicações de teste
- Workloads Kubernetes
- Serviços futuros do laboratório

---

# **Arquitetura Definida**

Modo de operação:

- Layer 2 (L2)

Não utilizar neste laboratório:

- BGP
- FRR
- Integração com roteadores
- ASN próprio

Objetivo:

Manter uma arquitetura simples, reproduzível e fácil de reconstruir.

---

# **Dependências**

Antes de iniciar:

- [[Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift\|05 - Instalação OpenShift]] concluído
- Cluster OpenShift operacional
- Rede validada
- Faixa IP reservada para LoadBalancers

Validação:

```bash
oc get nodes
oc get co
```

Resultado esperado:

- Todos os nodes Ready
- Todos os Cluster Operators Available
- Nenhum Cluster Operator Degraded

---

# **Planejamento de Rede**

Informações obrigatórias:

## **Rede**

- Gateway
- Máscara
- VLAN
- Subnet

## **Pool MetalLB**

Reservar uma faixa exclusiva para:

- Services LoadBalancer
- PX-Central
- PX-Backup
- Testes operacionais

Critérios:

- Não utilizar IPs distribuídos por DHCP
- Não utilizar IPs já reservados
- Não utilizar API VIP
- Não utilizar Ingress VIP
- Documentar a faixa utilizada

Exemplo:

```text
192.168.40.200-192.168.40.220
```

---

# **Documentação Oficial do Projeto**

Checklist:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/checklist_instalacao_metallb_openshift_4_21_15\|Arquivos/MetalLB/checklist_instalacao_metallb_openshift_4_21_15]]

Runbook:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/runbook_metallb_openshift_4_21_15\|Arquivos/MetalLB/runbook_metallb_openshift_4_21_15]]

Arquivos utilizados:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/IPAddressPool.yaml\|Arquivos/MetalLB/IPAddressPool.yaml]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/L2Advertisement.yaml\|Arquivos/MetalLB/L2Advertisement.yaml]]

---

# **Entregáveis da Etapa**

Ao final registrar:

## **Configuração**

- Namespace do MetalLB
- Nome do Pool
- Range IP configurado
- VLAN utilizada

## **Recursos**

- MetalLB CR
- IPAddressPool
- L2Advertisement

## **Validações**

- Service LoadBalancer funcional
- EXTERNAL-IP atribuído
- Conectividade validada

---

# **Resultado Esperado**

Validação:

```bash
oc get metallb -A
oc get ipaddresspool -A
oc get l2advertisement -A
oc get svc -A | grep LoadBalancer
```

Critérios mínimos:

- MetalLB instalado
- Controller saudável
- Speaker saudável
- Pool criado
- L2Advertisement criado
- Service LoadBalancer funcional

---

# **Uso Futuro no Lab POA**

Após esta etapa o cluster estará preparado para:

- [[Portworx/Portworx Labs/Lab POA/07 - Instalação Portworx\|07 - Instalação Portworx]]
- [[Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization\|08 - OpenShift Virtualization]]
- [[Portworx/Portworx Labs/Lab POA/09 - PX-Central\|09 - PX-Central]]
- [[Portworx/Portworx Labs/Lab POA/10 - PX-Backup\|10 - PX-Backup]]

---

# **Status**

Status atual:

- ☐ Não iniciado
- ☐ Em andamento
- ☐ Concluído

Data da instalação:

Responsável:

Observações: