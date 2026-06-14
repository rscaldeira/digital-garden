---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/runbook_openshift_virtualization_4_21/","dg-note-properties":{}}
---

# Runbook de instalação e configuração do OpenShift Virtualization 4.21 via Operator

## 1. Escopo deste runbook

Este runbook foi montado para OpenShift Container Platform 4.21, com instalação do OpenShift Virtualization via Operator no namespace obrigatório `openshift-cnv`, usando o fluxo padrão suportado pela documentação do produto.

Para o Lab POA, este runbook já assume dependências de ambiente que você definiu antes: Portworx saudável, MetalLB funcional e StorageClasses específicas para VMs, CDI e RWX.

## 2. Informações do ambiente

Preencha estes dados antes de começar:

```text
Cluster OpenShift: __________________
Versão OCP: 4.21.x
Namespace do OpenShift Virtualization: openshift-cnv
Portworx saudável: Sim / Não
StorageCluster Online: Sim / Não
Todos os nós Portworx Online: Sim / Não
Snapshot Controller instalado: Sim / Não
MetalLB instalado: Sim / Não
IPAddressPool criado: Sim / Não
L2Advertisement criado: Sim / Não
Service LoadBalancer funcional: Sim / Não

StorageClass padrão para VMs: px-vm-block-repl2
StorageClass para VMs de laboratório: px-vm-block-repl1
StorageClass CDI: px-cdi-scratch
StorageClass RWX: px-rwx-file-kubevirt
VolumeSnapshotClass para VMs: __________________
```

## 3. Variáveis sugeridas

```bash
export CNV_NS="openshift-cnv"
export VM_SC_DEFAULT="px-vm-block-repl2"
export VM_SC_LAB="px-vm-block-repl1"
export VM_SC_CDI="px-cdi-scratch"
export VM_SC_RWX="px-rwx-file-kubevirt"
```

## 4. Pré-check do cluster OpenShift

Antes da instalação, valide os pré-requisitos básicos do OpenShift Virtualization:

* o cluster deve estar em OpenShift 4.21;
* o usuário deve ter permissão `cluster-admin`;
* os worker nodes devem usar RHCOS; RHEL worker nodes não são suportados;
* as CPUs devem ser compatíveis com RHEL 9 e com extensões de virtualização habilitadas, como Intel VT-x ou AMD-V;
* se houver CPUs diferentes entre workers, pode haver falha de live migration, então isso deve ser avaliado antes.

Validação sugerida:

```bash
oc whoami
oc get clusterversion
oc get nodes -o wide
oc get mcp
```

Resultado esperado:

* usuário administrativo válido
* cluster disponível
* nós em `Ready`
* MCPs sem degradação

## 5. Pré-requisitos do Lab POA

### 5.1 Portworx

Antes de instalar o OpenShift Virtualization, valide que o backend Portworx já está pronto para atender discos de VM no ambiente.

Validação sugerida:

```bash
oc get storagecluster -A
oc get pods -n portworx
PX_POD=$(oc get pod -n portworx -l name=portworx -o jsonpath='{.items[0].metadata.name}')
oc exec -n portworx -it ${PX_POD} -c portworx -- /opt/pwx/bin/pxctl status
oc exec -n portworx -it ${PX_POD} -c portworx -- /opt/pwx/bin/pxctl cluster list
```

Resultado esperado:

* `StorageCluster` online
* todos os nós Portworx online
* cluster Portworx saudável

### 5.2 Snapshot Controller

Se o provisionador suportar snapshots, a documentação exige associar uma `VolumeSnapshotClass` à StorageClass padrão usada pelas VMs.

Validação sugerida:

```bash
oc get volumesnapshotclass
oc get pods -A | grep -i snapshot
```

### 5.3 MetalLB

Como o Lab POA já usa `LoadBalancer` para outros componentes e para cenários futuros com VMs, valide que o MetalLB já está operacional antes da instalação do OpenShift Virtualization.

Validação sugerida:

```bash
oc get metallb -A
oc get ipaddresspool -A
oc get l2advertisement -A
oc get svc -A | grep LoadBalancer
```

Resultado esperado:

* MetalLB presente
* pool criado
* L2Advertisement criado
* pelo menos um `Service LoadBalancer` funcional

## 6. Pré-check de storage para virtualização

A documentação exige storage suportado pelo OpenShift, com uma StorageClass default para virtualização, e recomenda associar snapshots quando o provisionador suportar isso.

No Lab POA, valide explicitamente:

* `px-vm-block-repl2` como StorageClass padrão para VMs
* `px-vm-block-repl1` para VMs de laboratório
* `px-cdi-scratch` para CDI
* `px-rwx-file-kubevirt` para casos específicos de RWX

Validação sugerida:

```bash
oc get sc
oc get sc ${VM_SC_DEFAULT} -o yaml
oc get sc ${VM_SC_LAB} -o yaml
oc get sc ${VM_SC_CDI} -o yaml
oc get sc ${VM_SC_RWX} -o yaml
oc get volumesnapshotclass
oc get storageprofile
```

Confirmações operacionais:

* `px-vm-block-repl2` definida para o uso principal de VMs
* `px-cdi-scratch` disponível para importações
* `px-rwx-file-kubevirt` mantida apenas para casos específicos
* `VolumeSnapshotClass` disponível para a classe usada com VMs, se o backend suportar snapshots
* `StorageProfile` presente para `px-vm-block-repl2`
* `StorageProfile` presente para `px-vm-block-repl1`
* `StorageProfile` presente para `px-cdi-scratch`
* `StorageProfile` presente para `px-rwx-file-kubevirt`

## 7. Planejamento de capacidade

O OpenShift Virtualization adiciona overhead de CPU, memória e storage ao cluster, e isso deve ser considerado antes de criar VMs.

Como referência do produto:

* nós de infraestrutura devem considerar capacidade para 4 cores adicionais distribuídos entre eles;
* workers que hospedam VMs devem considerar pelo menos 2 cores adicionais para os componentes de virtualização;
* o impacto estimado de storage é de cerca de 10 GiB por nó para a instalação do OpenShift Virtualization.

## 8. Instalação do Operator pela web console

Este runbook usa o caminho principal pela web console.

### Etapa 1 – Abrir o OperatorHub

Na perspectiva Administrator, abra o fluxo de instalação do OpenShift Virtualization Operator.

### Etapa 2 – Selecionar namespace e canal

Na instalação do Operator:

* mantenha o namespace recomendado, que instala em `openshift-cnv`;
* use `Automatic` como Approval Strategy;
* use o canal `stable` para instalar a versão compatível com o OpenShift 4.21.

A documentação alerta que instalar em namespace diferente de `openshift-cnv` faz a instalação falhar.

### Etapa 3 – Instalar e aguardar

Conclua a instalação e aguarde até que os pods do OpenShift Virtualization estejam em `Running`.

Validação sugerida:

```bash
oc get pods -n ${CNV_NS}
oc get csv -n ${CNV_NS}
```

Resultado esperado:

* pods em `Running`
* CSV em `Succeeded`

## 9. Criar e validar o HyperConverged

O recurso `HyperConverged` deve existir com o nome `kubevirt-hyperconverged` no namespace `openshift-cnv`.

Se precisar aplicar por CLI, use:

```yaml
apiVersion: hco.kubevirt.io/v1beta1
kind: HyperConverged
metadata:
  name: kubevirt-hyperconverged
  namespace: openshift-cnv
spec: {}
```

Aplicação:

```bash
oc apply -f hyperconverged.yaml
```

Validação rápida de troubleshooting:

```bash
oc get hco -n ${CNV_NS}
```

Resultado esperado:

* `Available=True`
* `Progressing=False`
* `Degraded=False`

Validação detalhada:

```bash
oc get hco -n ${CNV_NS} kubevirt-hyperconverged -o json | jq .status.versions
oc get hco kubevirt-hyperconverged -n ${CNV_NS} -o json | jq -r '.status.conditions[] | {type,status}'
```

Resultado esperado:

* `ReconcileComplete=True`
* `Available=True`
* `Progressing=False`
* `Degraded=False`
* `Upgradeable=True`

## 10. Validar CDI e componentes básicos

Depois do `HyperConverged`, valide CDI e os componentes centrais do stack.

Validação sugerida:

```bash
oc get pods -n ${CNV_NS}
oc get cdi
```

Resultado esperado:

* CDI Operator running
* CDI CR disponível
* `cdi-uploadproxy` running

## 11. Configuração inicial pós-instalação

### 11.1 StorageClass padrão de virtualização

A documentação informa que a StorageClass default de virtualização pode ser diferente da default geral do OpenShift, e a classe marcada com `storageclass.kubevirt.io/is-default-virt-class: "true"` terá precedência para discos de VM.

No Lab POA, valide que `px-vm-block-repl2` é a classe principal para VMs.

### 11.2 Live Migration

A documentação do OpenShift Virtualization trata storage compartilhado, recursos suficientes e CPUs compatíveis como pré-requisitos para live migration, e recomenda rede Multus dedicada para esse tráfego.

A documentação do OpenShift Virtualization/KubeVirt trata storage compartilhado como requisito padrão para Live Migration. Dependendo do backend de storage, podem existir implementações específicas que permitem migração utilizando volumes Block RWO.

No caso do Portworx, o comportamento deve ser validado na StorageClass utilizada pelas VMs antes de assumir suporte pleno para Live Migration.

No Lab POA, trate isso assim:

* validar que a StorageClass usada pelas VMs suporta o comportamento esperado de live migration conforme o backend Portworx
* validar com o time Portworx o comportamento esperado da `px-vm-block-repl2`
* executar teste prático com VM não crítica antes de assumir suporte pleno

### 11.3 Templates e imports do Lab POA

Depois da instalação, valide os artefatos operacionais do seu ambiente:

* Fedora VM template disponível
* Windows VM template disponível
* CDI import validado

## 12. Validação funcional inicial

### 12.1 Confirmar disponibilidade da interface

A documentação diz que, após todos os pods estarem em `Running`, o OpenShift Virtualization já pode ser usado pela interface Workloads.

### 12.2 Criar VMs de teste

No Lab POA, valide pelo menos:

* uma Fedora VM
* uma Windows VM

Critérios mínimos dessa etapa:

* console acessível
* disco provisionado via Portworx
* boot concluído com sucesso

### 12.3 Testar Live Migration

Antes do teste, valide que o utilitário `virtctl` está disponível na estação administrativa:

```bash
virtctl version
```

Valide a migração de forma prática:

* validar que Live Migration está habilitada
* executar migração de VM entre nós
* medir o comportamento da VM durante a migração
* registrar o resultado:
  * migração sem interrupção perceptível
  * interrupção breve durante o cutover
  * falha de migração
* validar conectividade da VM após a migração

Se você tiver `virtctl` disponível, pode usar:

```bash
virtctl migrate <vm-name>
```

### 12.4 Testar drain de node

Como parte do Lab POA, execute um drain controlado em um nó com VM:

```bash
oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data --force
```

Valide:

* comportamento da VM durante o drain
* ocorrência ou não de live migration automática
* ausência de downtime perceptível, quando aplicável
* estado saudável do cluster após a ação

Depois disso, execute o retorno do nó:

```bash
oc adm uncordon <node-name>
```

E confirme o retorno ao estado saudável com:

```bash
oc get nodes
oc get co
oc get pods -n ${CNV_NS}
```

### 12.5 Testar snapshot e clone

No Lab POA, valide também:

* snapshot de VM funcional
* clone de VM funcional

## 13. Critérios de aceite finais

Considere este runbook concluído quando estes itens estiverem OK:

* Operator instalado em `openshift-cnv`
* canal `stable` selecionado
* Approval Strategy em `Automatic`
* CSV em `Succeeded`
* `HyperConverged` presente e saudável
* CDI Operator running
* CDI CR disponível
* `cdi-uploadproxy` running
* Portworx saudável e entregando storage para VMs
* MetalLB saudável no cluster
* `px-vm-block-repl2` validada para uso principal de VMs
* Fedora VM criada
* Windows VM criada
* console acessível
* disco provisionado via Portworx
* live migration validada
* snapshot validado
* clone validado


---

## Sources

- [Chapter 4. Installing | Virtualization product documentation | OpenShift Container Platform | 4.21 | Red Hat Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/virtualization/installing)