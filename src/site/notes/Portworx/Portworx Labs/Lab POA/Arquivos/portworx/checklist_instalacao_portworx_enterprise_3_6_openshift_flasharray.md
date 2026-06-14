---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/portworx/checklist_instalacao_portworx_enterprise_3_6_openshift_flasharray/","dg-note-properties":{}}
---

# Checklist de instalação e configuração do Portworx Enterprise 3.6 no OpenShift 4.21.15

## Escopo

* [ ] Este checklist assume um ambiente OpenShift 4.21.15 já disponível, com MetalLB já implantado no cluster, e Portworx Enterprise 3.6 sendo instalado com FlashArray, usando Portworx Operator e Portworx Central para gerar as especificações do Operator e do `StorageCluster`.
* [ ] Confirmar que o método escolhido é o fluxo Portworx Central-based installation, no qual o Portworx Central gera uma especificação customizada para o seu storage platform e distribuição Kubernetes/OpenShift.
* [ ] Confirmar que o backend de storage será FlashArray, que suporta cloud drives, direct access volumes, PX-StoreV2, secure multi-tenancy e ActiveCluster, conforme o desenho do ambiente.

## 1. Pré-requisitos de plataforma

* [ ] Validar que o cluster Portworx terá pelo menos 3 nós.
* [ ] Confirmar que cada nó atende aos requisitos mínimos de hardware para o modelo de storage que será usado:
  * PX-StoreV1: mínimo 4 cores, 4 GB RAM, `/opt` com pelo menos 3.5 GB livres e root filesystem mínimo de 64 GB.
  * PX-StoreV2: mínimo 8 cores, 8 GB RAM, `/opt` com pelo menos 3.5 GB livres, `/var` recomendado com 20 GB livres e root filesystem mínimo de 64 GB quando `/opt` e `/var` forem separados; caso contrário, 128 GB.
* [ ] Confirmar que os discos usados pelo Portworx são block devices desmontados, como raw disks, partições, LVM ou cloud block storage.
* [ ] Validar kernel suportado para o modelo escolhido:
  * PX-StoreV1: kernel 4.18 ou superior.
  * PX-StoreV2: kernel 4.20 ou superior, com 5.0 ou superior recomendado.
* [ ] Desabilitar swap em todos os nós e garantir que ela não volte após reboot.
* [ ] Garantir sincronismo NTP em todos os nós.
* [ ] Definir se o cluster usará Internal KVDB ou KVDB externo; o produto suporta ambos.

## 2. Pré-requisitos de rede e firewall

* [ ] Validar largura de banda mínima de 1 Gbps entre nós, com 10 Gbps recomendado.
* [ ] Validar latência menor que 10 ms entre nós para replicação síncrona.
* [ ] Validar as portas internas exigidas no OpenShift para Portworx, incluindo 17001, 17002/UDP, 17003, 17004, 17009, 17010, 17011, 17015, 17016, 17017, 17018, 17019 e 17021, além de 20001 e 20002 quando aplicável.
* [ ] Validar portas inbound mínimas no OpenShift para o gateway/management do Portworx, como 17001 e 17018.
* [ ] Validar saída TCP/443 para instalação, downloads, licenciamento e telemetry, incluindo `install.portworx.com` e `mirrors.portworx.com`.

## 3. Preparação específica do FlashArray

* [ ] Confirmar que o FlashArray será usado como backend do Portworx.
* [ ] Garantir que cada nó tenha acesso ao management IP do FlashArray.
* [ ] Se o ambiente usar iSCSI, confirmar que os iniciadores iSCSI dos nós estão na mesma VLAN dos target ports do FlashArray.
* [ ] Se o ambiente usar Fibre Channel, confirmar zoning correto entre os WWNs dos nós e as portas FC do FlashArray.
* [ ] Preparar as credenciais do FlashArray para uso no Portworx, normalmente management endpoint e API token, para alimentar a configuração gerada no Portworx Central.
* [ ] Se houver múltiplos FlashArrays e necessidade de controle por topologia, planejar labels de nós e labels dos arrays antes da instalação.

## 3.1 MachineConfigs obrigatórias

* [ ] Aplicar MachineConfig de Multipath.
* [ ] Aplicar MachineConfig de Udev Rules.
* [ ] Validar criação ou atualização dos MachineConfigPools.
* [ ] Aguardar os MCPs chegarem em `Updated=True`.
* [ ] Confirmar reboot dos nós após a aplicação, quando exigido pelo OpenShift.
* [ ] Validar o estado final com:

```bash
oc get mcp
oc get machineconfig
```

## 3.2 Preparação do Pure Secret

* [ ] Gerar o arquivo `pure.json` com os parâmetros exigidos para integração com o FlashArray.
* [ ] Validar o endpoint de management do FlashArray.
* [ ] Validar o API Token antes de criar o secret.
* [ ] Criar o secret `px-pure-secret` no namespace que será usado pelo Portworx.
* [ ] Validar a existência do secret antes de aplicar o `StorageCluster` com:

```bash
oc get secret px-pure-secret -A
```

## 3.3 SSD local para metadata, se aplicável

* [ ] Confirmar existência de SSD local para metadata, se esse for o desenho adotado.
* [ ] Confirmar que o device está em modo RAW.
* [ ] Confirmar que o device está visível no host.
* [ ] Confirmar que o device foi selecionado corretamente no Spec Generator.
* [ ] Se a metadata ficar no FlashArray e não em SSD local, marcar este item como N/A.

## 4. Preparação do OpenShift e do Portworx Central

* [ ] Confirmar acesso administrativo ao OpenShift via `oc`.
* [ ] Confirmar acesso ao Portworx Central.
* [ ] Confirmar que a opção escolhida é o fluxo com especificação gerada pelo Portworx Central para o seu ambiente OpenShift + FlashArray.
* [ ] Definir previamente o namespace do Portworx Operator e do `StorageCluster`.
* [ ] Definir previamente o nome do cluster Portworx que aparecerá no `StorageCluster`.
* [ ] Definir se o deployment será com FlashArray cloud drives ou direct access volumes.

## 5. Gerar a especificação no Portworx Central

* [ ] No Portworx Central, selecionar OpenShift como plataforma de destino e FlashArray como storage platform.
* [ ] Preencher os parâmetros do ambiente e gerar a especificação do Operator e do `StorageCluster`.
* [ ] Confirmar que a especificação final contempla a configuração do FlashArray e do modelo de deployment desejado.
* [ ] Confirmar PX-StoreV2 habilitado no `StorageCluster`.
* [ ] Confirmar o tamanho dos cloud drives, se esse for o modelo adotado.
* [ ] Confirmar o Metadata Device configurado.
* [ ] Confirmar o Journal Device conforme o padrão definido para PX-StoreV2.
* [ ] Confirmar o número de storage nodes que participarão do cluster Portworx.
* [ ] Se for usar secure multi-tenancy com FlashArray, revisar a seção `cloudStorage` e o placement de system volumes nos pods do FlashArray conforme o desenho do ambiente.

## 6. Instalar o Portworx Operator

* [ ] Instalar primeiro o Portworx Operator.
* [ ] Se optar pelo fluxo OpenShift UI, ir em OperatorHub, buscar `Portworx Operator` e instalar no namespace desejado.
* [ ] Se optar pelo fluxo gerado pelo Portworx Central, copiar o comando `oc apply -f '<url-generated-from-portworx-central-spec-gen>'` referente ao Operator e aplicá-lo.
* [ ] Validar que os recursos do Operator foram criados com sucesso, como namespace, serviceaccount, clusterrole, clusterrolebinding e deployment do operator.

## 7. Criar o StorageCluster

* [ ] Aplicar a especificação do `StorageCluster` gerada pelo Portworx Central somente após o Operator estar pronto.
* [ ] Confirmar que o `StorageCluster` foi criado com sucesso.
* [ ] Garantir que a configuração final referencia corretamente o FlashArray como storage provider.

## 8. Validações pós-deploy

* [ ] Validar que o Portworx detectou aconfiguração do FlashArray e iniciou usando o array como storage provider.
* [ ] Validar que os pods do Portworx estão em estado saudável no namespace escolhido.
* [ ] Validar que o `StorageCluster` está `Online` ou equivalente no status.
* [ ] Validar que os nós esperados participaram do cluster Portworx.
* [ ] Validar que não há falhas de conectividade entre nós, nem erros de acesso ao FlashArray.
* [ ] Executar validação operacional real do Portworx com:

```bash
pxctl status
pxctl cluster list
pxctl service pool show
```

* [ ] Resultado esperado:
  * Cluster Online
  * Todos os nós Online
  * Pools Online

## 9. Configuração inicial de storage para workloads

* [ ] Aplicar as `StorageClasses` padrão do ambiente.
* [ ] Validar a `StorageClass` `px-app-repl1`.
* [ ] Validar a `StorageClass` `px-app-repl2`.
* [ ] Validar a `StorageClass` `px-app-repl3`.
* [ ] Validar a `StorageClass` `px-vm-block-repl1`.
* [ ] Validar a `StorageClass` `px-vm-block-repl2`.
* [ ] Validar a `StorageClass` `px-rwx-file-kubevirt`.
* [ ] Validar a `StorageClass` `px-cdi-scratch`.
* [ ] Testar uma PVC em uma `StorageClass` Portworx para comprovar provisionamento funcional no ambiente.
* [ ] Se o ambiente usar FlashArray topology, validar labels de nós, labels dos arrays e `StorageClass` com `allowedTopologies` ou `WaitForFirstConsumer`, conforme o desenho.

## 10. Considerações específicas do ambiente com MetalLB

* [ ] Confirmar que o MetalLB já está operacional no OpenShift antes de iniciar integrações que dependam de `Service LoadBalancer`.
* [ ] Confirmar que o Portworx Enterprise em si não depende do MetalLB para criação do `StorageCluster`, mas que componentes adicionais do ecossistema podem depender de `LoadBalancer` no pós-instalação.
* [ ] Validar que o cluster permanece pronto para uso conjunto com PX-Central e PX-Backup após a instalação do Portworx.

## 11. Critérios de aceite

* [ ] Pré-requisitos de hardware, software e rede validados.
* [ ] Conectividade e credenciais do FlashArray preparadas.
* [ ] Especificação do Operator gerada no Portworx Central.
* [ ] Portworx Operator instalado com sucesso.
* [ ] Especificação do `StorageCluster` gerada no Portworx Central.
* [ ] `StorageCluster` criado com sucesso.
* [ ] Portworx detectando o FlashArray como backend.
* [ ] Pelo menos uma `StorageClass` Portworx disponível para workloads.
* [ ] Teste de PVC concluído com sucesso.
* [ ] Ambiente pronto para receber PX-Central, PX-Backup e workloads persistentes.


---

## Sources

- [Install Portworx Enterprise | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/install)
- [Installation of Portworx with FlashArray | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/install/pure-storage/flasharray)
- [Installation of Portworx with FlashArray using Portworx Central ...](https://docs.portworx.com/portworx-enterprise/platform/install/pure-storage/flasharray/install-flasharray)
- [Install Portworx with Everpure Platforms | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/install/pure-storage)
- [System Requirements | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/prerequisites)
- [Disk provisioning on Pure Storage FlashArray](https://docs.portworx.com/portworx-enterprise/cloud-references/auto-disk-provisioning/pure-flash-array)
- [CSI topology for FlashArray cloud drives - Portworx Documentation](https://docs.portworx.com/portworx-enterprise/operations/operate-kubernetes/cluster-topology/csi-topology-cloud-drives)
- [CSI topology for FlashArray Direct Access volumes and FlashBlade ...](https://docs.portworx.com/portworx-enterprise/how-to-guides/csi-topology)