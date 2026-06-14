---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/virtualization/checklist_instalacao_openshift_virtualization_4_21/","dg-note-properties":{}}
---

# Checklist de instalação e configuração do OpenShift Virtualization 4.21 via Operator

## Escopo

* [ ] Este checklist cobre a instalação do OpenShift Virtualization em um cluster OpenShift Container Platform 4.21 usando Operator, com foco no fluxo padrão suportado pela documentação do produto.
* [ ] O método de instalação é o OpenShift Virtualization Operator no namespace obrigatório `openshift-cnv`.

## 1. Pré-requisitos do cluster

* [ ] Confirmar que o cluster já está em OpenShift Container Platform 4.21.
* [ ] Confirmar acesso com usuário `cluster-admin` no cluster e na web console.
* [ ] Confirmar que o `oc` está instalado, caso também queira validar ou instalar por CLI.
* [ ] Validar CPU compatível com RHEL 9 e com extensões de virtualização habilitadas, como Intel VT-x, AMD-V ou equivalentes da arquitetura suportada.
* [ ] Confirmar que os worker nodes usam RHCOS. RHEL worker nodes não são suportados para OpenShift Virtualization.
* [ ] Validar se os worker nodes que hospedarão VMs têm CPUs compatíveis entre si, para evitar problemas de live migration.

## 2. Pré-requisitos de storage e operação

* [ ] Confirmar que o Portworx está instalado e saudável.
* [ ] Confirmar que o `StorageCluster` está `Online`.
* [ ] Confirmar que todos os nós Portworx estão `Online`.
* [ ] Confirmar que o Snapshot Controller está instalado.
* [ ] Validar a StorageClass padrão para VMs:
  * [ ] `px-vm-block-repl2`
* [ ] Validar a StorageClass para VMs de laboratório:
  * [ ] `px-vm-block-repl1`
* [ ] Validar a StorageClass CDI:
  * [ ] `px-cdi-scratch`
* [ ] Validar a StorageClass RWX:
  * [ ] `px-rwx-file-kubevirt`
* [ ] Definir a StorageClass padrão para virtualização usando a annotation `storageclass.kubevirt.io/is-default-virt-class: "true"` quando necessário.
* [ ] Se o provisionador suportar snapshots, associar uma `VolumeSnapshotClass` à StorageClass padrão usada pelas VMs.
* [ ] Validar que a StorageClass utilizada pelas VMs suporta Live Migration conforme o backend de storage utilizado.
* [ ] VMs com volumes RWO (Block) podem apresentar restrições de Live Migration dependendo da implementação do storage.
* [ ] Validar com o time Portworx o comportamento esperado da StorageClass `px-vm-block-repl2`.
* [ ] Executar teste prático de Live Migration com uma VM não crítica antes de assumir suporte pleno.
* [ ] Confirmar que o MetalLB está instalado.
* [ ] Confirmar que existe um `IPAddressPool` criado.
* [ ] Confirmar que existe um `L2Advertisement` criado.
* [ ] Confirmar que `Service LoadBalancer` já foi validado no cluster.

## 3. Planejamento de capacidade

* [ ] Reservar overhead adicional de memória, CPU e storage no cluster para o OpenShift Virtualization antes de colocar VMs em produção.
* [ ] Considerar pelo menos 4 cores adicionais distribuídos nos nós de infraestrutura e pelo menos 2 cores adicionais nos worker nodes que hospedarão VMs.
* [ ] Considerar aproximadamente 10 GiB de impacto de storage por nó para a instalação do OpenShift Virtualization.

## 4. Instalação do Operator pela web console

* [ ] Acessar a web console no perfil Administrator e abrir o fluxo de instalação do Operator.
* [ ] No install do Operator, selecionar o namespace recomendado. O Operator deve ser instalado em `openshift-cnv`; instalar em outro namespace faz a instalação falhar.
* [ ] Definir `Automatic` como Approval Strategy. A documentação recomenda isso para manter compatibilidade e suporte com a versão correspondente do OpenShift.
* [ ] Selecionar o canal `stable` para instalar a versão compatível do OpenShift Virtualization com o OpenShift 4.21.
* [ ] Aguardar a instalação do Operator.

## 5. Verificação do Operator

* [ ] Validar que os pods do OpenShift Virtualization no namespace `openshift-cnv` chegaram a `Running`.
* [ ] Validar que o CSV do Operator está em `Succeeded`.

```bash
oc get csv -n openshift-cnv
```

## 6. Configuração e deploy do HyperConverged

* [ ] Confirmar que o recurso `HyperConverged` foi criado com o nome `kubevirt-hyperconverged` no namespace `openshift-cnv`.
* [ ] Se necessário instalar por CLI, aplicar o manifest do `HyperConverged` após a subscription do catálogo já estar pronta.

Exemplo de criação por CLI:

```yaml
apiVersion: hco.kubevirt.io/v1beta1
kind: HyperConverged
metadata:
  name: kubevirt-hyperconverged
  namespace: openshift-cnv
spec: {}
```

## 7. Verificações do HyperConverged

* [ ] Validar rapidamente o estado do `HyperConverged`.

```bash
oc get hco -n openshift-cnv
```

Resultado esperado:

* `Available=True`
* `Progressing=False`
* `Degraded=False`

* [ ] Validar a versão do `HyperConverged` instalada.

```bash
oc get hco -n openshift-cnv kubevirt-hyperconverged -o json | jq .status.versions
```

* [ ] Validar as condições detalhadas do `HyperConverged`.

```bash
oc get hco kubevirt-hyperconverged -n openshift-cnv -o json | jq -r '.status.conditions[] | {type,status}'
```

Resultado esperado:

* `ReconcileComplete=True`
* `Available=True`
* `Progressing=False`
* `Degraded=False`
* `Upgradeable=True`

## 8. Configuração inicial pós-instalação

* [ ] Validar se a StorageClass padrão para virtualização está definida.
* [ ] Validar se a `VolumeSnapshotClass` está associada ao storage padrão das VMs, caso snapshots sejam suportados pelo provisionador.
* [ ] Definir quais worker nodes hospedarão workloads de virtualização e revisar afinidade de nós se houver CPUs heterogêneas.
* [ ] Se o ambiente usará live migration, validar storage compartilhado e capacidade de memória/rede suficiente no cluster.
* [ ] Se possível, planejar uma rede dedicada Multus para live migration, pois isso é altamente recomendado pela documentação.
* [ ] Validar o CDI após a criação do `HyperConverged`.
* [ ] Validar que o CDI Operator está em `Running`.
* [ ] Validar que o recurso `CDI` está `Available`.
* [ ] Validar que o `CDI UploadProxy` está em `Running`.

Validação sugerida:

```bash
oc get pods -n openshift-cnv
oc get cdi
```

## 9. Templates e artefatos iniciais

* [ ] Validar que o template de Fedora VM está disponível.
* [ ] Validar que o template de Windows VM está disponível.
* [ ] Validar o fluxo de CDI Import.

## 10. Validação funcional inicial

* [ ] Acessar Workloads e confirmar que a interface de virtualização está disponível após a instalação do Operator.
* [ ] Confirmar que o OpenShift Virtualization está utilizável depois que todos os pods estiverem em `Running`.
* [ ] Validar que o ambiente já está apto para criação de VMs com storage compatível e classe default definida.
* [ ] Criar uma Fedora VM.
* [ ] Criar uma Windows VM.
* [ ] Validar que o console das VMs está acessível.
* [ ] Validar que o disco foi provisionado via Portworx.
* [ ] Validar que Live Migration está habilitada.
* [ ] Executar migração de VM entre nós.
* [ ] Medir comportamento da VM durante a migração.
* [ ] Registrar o resultado:
  * [ ] Migração sem interrupção perceptível
  * [ ] Interrupção breve durante o cutover
  * [ ] Falha de migração
* [ ] Validar conectividade da VM após a migração.
* [ ] Executar drain de node contendo VM.
* [ ] Validar comportamento da VM durante o drain.
* [ ] Validar se houve Live Migration automática.
* [ ] Validar ausência de downtime perceptível.
* [ ] Executar uncordon do node.
* [ ] Confirmar retorno do cluster ao estado saudável.
* [ ] Validar snapshot de VM funcional.
* [ ] Validar clone de VM funcional.

## 11. Critérios de aceite

* [ ] Operator instalado em `openshift-cnv`.
* [ ] Canal `stable` selecionado.
* [ ] Approval Strategy em `Automatic`.
* [ ] CSV do Operator em `Succeeded`.
* [ ] Pods do OpenShift Virtualization em `Running`.
* [ ] `HyperConverged` presente no namespace `openshift-cnv`.
* [ ] `HyperConverged` com `Available=True`, `Degraded=False` e `Progressing=False`.
* [ ] CDI Operator em `Running`.
* [ ] CDI `Available`.
* [ ] StorageClass default de virtualização definida.
* [ ] Fedora VM criada.
* [ ] Windows VM criada.
* [ ] Console acessível.
* [ ] Disco provisionado via Portworx.
* [ ] Live Migration validada.
* [ ] Snapshot validado.
* [ ] Clone validado.
* [ ] Cluster pronto para criação e operação inicial de VMs no OpenShift Virtualization.


---

## Sources

- [Chapter 4. Installing | Virtualization product documentation | OpenShift Container Platform | 4.21 | Red Hat Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/virtualization/installing)