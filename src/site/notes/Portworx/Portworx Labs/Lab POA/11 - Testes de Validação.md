---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/11 - Testes de Validação/","dg-note-properties":{}}
---


# **Testes de Validação**

## **Objetivo**

Este documento consolida todos os testes de validação do Lab POA.

O objetivo é validar que todos os componentes implantados estão operacionais e que a integração entre OpenShift, MetalLB, FlashArray, Portworx, OpenShift Virtualization, PX-Central e PX-Backup funciona conforme esperado.

Este documento deve ser executado após a conclusão das etapas de instalação do ambiente.

---

# **Dependências**

Este documento deve ser executado somente após:

- [[Portworx/Portworx Labs/Lab POA/05 - Instalação OpenShift\|05 - Instalação OpenShift]]
- [[Portworx/Portworx Labs/Lab POA/06 - Instalação MetalLB\|06 - Instalação MetalLB]]
- [[Portworx/Portworx Labs/Lab POA/07 - Instalação Portworx\|07 - Instalação Portworx]]
- [[Portworx/Portworx Labs/Lab POA/08 - Openshift Virtualization\|08 - OpenShift Virtualization]]
- [[Portworx/Portworx Labs/Lab POA/09 - PX-Central\|09 - PX-Central]]

Opcional:

- [[Portworx/Portworx Labs/Lab POA/10 - PX-Backup\|10 - PX-Backup]]

---

# **Objetivos da Validação**

Validar:

- Saúde do cluster OpenShift
- Funcionamento do MetalLB
- Integração com FlashArray
- Saúde do Portworx
- Provisionamento de volumes
- Snapshots
- Clones
- OpenShift Virtualization
- Live Migration
- Node Drain
- PX-Central
- PX-Backup

---

# **1. OpenShift**

## **Validar Cluster Operators**

```bash
oc get co
```

Resultado esperado:

- Todos Available=True
- Todos Degraded=False

---

## **Validar Nodes**

```bash
oc get nodes
```

Resultado esperado:

- Todos Ready

---

## **Validar MachineConfigPools**

```bash
oc get mcp
```

Resultado esperado:

- Updated=True
- Updating=False
- Degraded=False

---

# **2. MetalLB**

## **Validar Controller**

```bash
oc get deployment -n metallb-system controller
```

Resultado esperado:

- Available=True

---

## **Validar Speaker**

```bash
oc get daemonset -n metallb-system speaker
```

Resultado esperado:

- Um speaker por node elegível

---

## **Validar Pools**

```bash
oc get ipaddresspool -A
oc get l2advertisement -A
```

Resultado esperado:

- Pool criado
- L2Advertisement criado

---

## **Validar Service LoadBalancer**

Criar um serviço de teste utilizando o procedimento definido em:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/metallb/runbook_metallb_openshift_4_21_15\|Arquivos/metallb/runbook_metallb_openshift_4_21_15]]

Resultado esperado:

- EXTERNAL-IP atribuído
- Acesso funcional

---

# **3. FlashArray**

## **Validar Backend Storage**

Validar:

- Conectividade dos nodes
- Cloud Drives criados
- Sem erros de comunicação

---

## **Validar Pure Secret**

```bash
oc get secret px-pure-secret -A
```

Resultado esperado:

- Secret existente

Documentos relacionados:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray Configuration\|Arquivos/flasharray/FlashArray Configuration]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - pure.json\|Arquivos/flasharray/FlashArray - pure.json]]
- [[Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - Secret px-pure-secret.yaml\|Arquivos/flasharray/FlashArray - Secret px-pure-secret.yaml]]

---

# **4. Portworx**

## **Validar StorageCluster**

```bash
oc get storagecluster -A
```

Resultado esperado:

- Online
- Healthy

---

## **Validar Cluster**

```bash
pxctl status
```

Resultado esperado:

- Cluster Online

---

## **Validar Nodes**

```bash
pxctl cluster list
```

Resultado esperado:

- Todos os nodes Online

---

## **Validar Pools**

```bash
pxctl service pool show
```

Resultado esperado:

- Todos os pools Online

---

# **5. StorageClasses**

Validar existência das classes:

```bash
oc get sc
```

Resultado esperado:

- px-app-repl1
- px-app-repl2
- px-app-repl3
- px-vm-block-repl1
- px-vm-block-repl2
- px-rwx-file-kubevirt
- px-cdi-scratch

Referência:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/Storage Classes Overview\|Arquivos/storageclasses/Storage Classes Overview]]

---

# **6. PVC Básico**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/PVC - Basic Test.yaml\|Arquivos/tests/PVC - Basic Test.yaml]]

Resultado esperado:

- PVC Bound
- PV criado
- StorageClass correta

---

# **7. PVC RWX**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/PVC - RWX Test.yaml\|Arquivos/tests/PVC - RWX Test.yaml]]

Resultado esperado:

- PVC Bound
- RWX funcional
- Montagem simultânea possível

---

# **8. Snapshot**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/Snapshot Test.yaml\|Arquivos/tests/Snapshot Test.yaml]]

Resultado esperado:

- Snapshot criada
- Snapshot utilizável para restore

---

# **9. Clone**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/Clone Test.yaml\|Arquivos/tests/Clone Test.yaml]]

Resultado esperado:

- Clone criado
- Clone acessível
- Clone íntegro

---

# **10. FIO**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/tests/FIO Test.yaml\|Arquivos/tests/FIO Test.yaml]]

Resultado esperado:

- Execução concluída
- Sem erros de I/O

Registrar:

- IOPS
- Throughput
- Latência

---

# **11. OpenShift Virtualization**

## **Validar HyperConverged**

```bash
oc get hco -n openshift-cnv
```

Resultado esperado:

- Available=True
- Degraded=False

---

## **Validar CDI**

```bash
oc get cdi
```

Resultado esperado:

- CDI Available

---

## **Validar Fedora VM**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Fedora VM.yaml\|Arquivos/virtualization/Fedora VM.yaml]]

Resultado esperado:

- VM criada
- Boot concluído
- Console acessível

---

## **Validar Windows VM**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/Windows VM.yaml\|Arquivos/virtualization/Windows VM.yaml]]

Resultado esperado:

- VM criada
- Boot concluído
- Console acessível

---

## **Validar CDI Import**

Artefato:

- [[Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/CDI Import.yaml\|Arquivos/virtualization/CDI Import.yaml]]

Resultado esperado:

- Import concluído
- DataVolume pronta

---

# **12. Live Migration**

Executar migração de VM entre nodes.

Resultado esperado:

- Migração concluída
- VM acessível após migração

Registrar:

- Sem interrupção perceptível
- Interrupção breve
- Falha

Observação:

O comportamento depende da combinação entre OpenShift Virtualization, KubeVirt, StorageClass utilizada e recursos Portworx habilitados.

---

# **13. Node Drain**

Executar:

```bash
oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data --force
```

Validar:

- Comportamento das VMs
- Migração automática
- Recuperação do cluster

Retorno:

```bash
oc adm uncordon <node-name>
```

Resultado esperado:

- Cluster saudável
- Nodes Ready

---

# **14. PX-Central**

Validar:

- Dashboard acessível
- Nodes visíveis
- Pools visíveis
- Volumes visíveis
- Capacity correta

Referência:

- [[Portworx/Portworx Labs/Lab POA/09 - PX-Central\|09 - PX-Central]]

---

# **15. PX-Backup (Opcional)**

Validar:

- Console acessível
- Cluster integrado
- Backup Location criada
- Backup funcional
- Restore funcional

---

## **Backup Kubernetes**

Validar:

- Backup de namespace
- Restore de namespace

---

## **Backup de VM**

Validar:

- Backup Fedora VM
- Backup Windows VM
- Restore funcional

Referência:

- [[Portworx/Portworx Labs/Lab POA/10 - PX-Backup\|10 - PX-Backup]]

---

# **Matriz Final de Aprovação**

|**Componente**|**Status**|
|---|---|
|OpenShift|☐|
|MetalLB|☐|
|FlashArray|☐|
|Portworx|☐|
|StorageClasses|☐|
|PVC Básico|☐|
|PVC RWX|☐|
|Snapshot|☐|
|Clone|☐|
|FIO|☐|
|OpenShift Virtualization|☐|
|Fedora VM|☐|
|Windows VM|☐|
|CDI Import|☐|
|Live Migration|☐|
|Node Drain|☐|
|PX-Central|☐|
|PX-Backup|☐|

---

# **Resultado Esperado**

Ao final deste documento o ambiente deve estar validado para demonstrações de:

- OpenShift
- OpenShift Virtualization
- VMware Replacement
- Portworx
- FlashArray
- Snapshots
- Clones
- Data Protection
- Backup e Restore
- Alta Disponibilidade
- Modernização de Infraestrutura

---

# **Status**

Status atual:

- ☐ Não iniciado
- ☐ Em andamento
- ☐ Concluído

Data da validação:

Responsável:

Observações: