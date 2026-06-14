---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/91 - Rebuild Time Estimate/","dg-note-properties":{}}
---


# **Rebuild Time Estimate**

## **Objetivo**

Este documento estima o tempo necessário para reconstrução completa do Lab POA a partir de servidores vazios até um ambiente totalmente validado.

Os tempos abaixo assumem:

- Infraestrutura física já montada
- FlashArray já operacional
- DNS disponível
- DHCP disponível
- Internet disponível
- Operador com conhecimento básico do ambiente

Os tempos representam uma expectativa realista, incluindo espera por reboots, reconciliação de Operators e validações.

---

# **Fase 1 - OpenShift**

|**Atividade**|**Tempo**|
|---|---|
|Criar cluster no Assisted Installer|10 min|
|Gerar Discovery ISO|5 min|
|Boot dos 3 servidores|15 min|
|Discovery dos hosts|10 min|
|Configuração final da instalação|10 min|
|Instalação do OpenShift|45–60 min|
|Validações iniciais|10 min|

**Tempo estimado:** 1h45 a 2h00

---

# **Fase 2 - Configurações Base do OpenShift**

|**Atividade**|**Tempo**|
|---|---|
|Cluster Monitoring Config|5 min|
|MachineConfigs|5 min|
|Reboot dos nós via MCP|20–30 min|
|Validação dos MCPs|5 min|

**Tempo estimado:** 35 a 45 min

---

# **Fase 3 - MetalLB**

|**Atividade**|**Tempo**|
|---|---|
|Instalação do Operator|5 min|
|Criação do MetalLB|5 min|
|IPAddressPool|2 min|
|L2Advertisement|2 min|
|Teste LoadBalancer|5 min|

**Tempo estimado:** 15 a 20 min

---

# **Fase 4 - Portworx**

|**Atividade**|**Tempo**|
|---|---|
|Criar px-pure-secret|5 min|
|Gerar StorageCluster no Spec Generator|10 min|
|Instalar Operator|10 min|
|Criar StorageCluster|5 min|
|Provisionamento inicial dos pools|10–15 min|
|Formação do cluster Portworx|10 min|
|Validação do pxctl|10 min|
|Aplicação das StorageClasses|5 min|
|Teste PVC|5 min|

**Tempo estimado:** 1h00

---

# **Fase 5 - OpenShift Virtualization**

|**Atividade**|**Tempo**|
|---|---|
|Instalar Operator|10 min|
|Criar HyperConverged|5 min|
|Deploy dos componentes|10 min|
|Validação CDI|5 min|
|Configuração inicial|5 min|

**Tempo estimado:** 30 a 40 min

---

# **Fase 6 - PX-Central**

|**Atividade**|**Tempo**|
|---|---|
|Gerar spec|10 min|
|Deploy|15 min|
|Inicialização dos serviços|10 min|
|Publicação via MetalLB|5 min|
|Login inicial|5 min|

**Tempo estimado:** 40 a 45 min

---

# **Fase 7 - PX-Backup**

|**Atividade**|**Tempo**|
|---|---|
|Gerar spec|10 min|
|Deploy|15 min|
|Inicialização dos serviços|15 min|
|Publicação via MetalLB|5 min|
|Login inicial|5 min|

**Tempo estimado:** 45 a 50 min

---

# **Fase 8 - Validações Funcionais**

|**Atividade**|**Tempo**|
|---|---|
|PVC Test|5 min|
|Snapshot Test|5 min|
|Clone Test|5 min|
|FIO Test|15 min|
|Fedora VM|10 min|
|Windows VM|15 min|
|Live Migration|10 min|
|Node Drain|10 min|
|Backup Test|10 min|
|Restore Test|10 min|

**Tempo estimado:** 1h30

---

# **Tempo Total Estimado**

|**Cenário**|**Tempo**|
|---|---|
|Melhor caso|~6h|
|Realista|~7h|
|Com troubleshooting leve|~8h|

---

# **Observações**

- O maior consumidor de tempo é a instalação inicial do OpenShift e os reboots provocados por MachineConfigs.
- Portworx normalmente fica operacional rapidamente após o StorageCluster, mas a validação completa dos pools deve ser aguardada.
- PX-Central e PX-Backup costumam consumir mais tempo na inicialização dos componentes do que na instalação em si.
- O ambiente só deve ser considerado concluído após a Validation Matrix estar 100% validada.