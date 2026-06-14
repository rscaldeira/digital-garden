---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/portworx/runbook_portworx_enterprise_3_6_openshift_4_21_15_flasharray/","dg-note-properties":{}}
---

# Runbook de instalação e configuração do Portworx Enterprise 3.6 no OpenShift 4.21.15 com FlashArray

## 1. Escopo deste runbook

Este runbook foi montado para um ambiente OpenShift 4.21.15 já operacional, com MetalLB já instalado, FlashArray como backend de storage e Portworx Enterprise 3.6 implantado via Portworx Operator, usando o Portworx Central para gerar a especificação do `StorageCluster`.

O fluxo oficial para FlashArray com Portworx Central é: preparar o ambiente, gerar a especificação no Portworx Central, instalar primeiro o Operator e depois aplicar o `StorageCluster`.

## 2. Informações do ambiente

Preencha estes dados antes de começar:

```text
Cluster OpenShift: __________________
Versão OCP: 4.21.15
Namespace do Portworx: __________________
Nome do StorageCluster: __________________
FlashArray Mgmt Endpoint: __________________
SAN Protocol: iSCSI / FC / NVMe-oF TCP / NVMe-oF RDMA
PX-Store: PX-StoreV2
Pool drive size (GB): __________________
Metadata device local: Sim / Não
Journal device: Auto / Custom / None
MetalLB validado: Sim / Não
Storage nodes planejados: __________________
```

## 3. Variáveis sugeridas

```bash
export PX_NS="portworx"
export PX_CLUSTER_NAME="px-cluster"
export PX_OPERATOR_NS="${PX_NS}"
export PX_PURE_SECRET_NAME="px-pure-secret"
export PX_SAN_TYPE="ISCSI"
export PX_STORE_TYPE="PX-StoreV2"
export PX_POOL_SIZE_GB="2000"
```

## 4. Pré-check de plataforma

Valide antes de qualquer aplicação:

* O cluster Portworx deve ter pelo menos 3 nós.
* Para PX-StoreV2, cada nó deve atender aos requisitos mínimos de CPU, RAM, espaço em disco e kernel; o mínimo documentado inclui kernel 4.20+ e 5.0+ recomendado.
* Os dispositivos usados pelo Portworx devem ser block devices desmontados.
* Swap deve estar desabilitado e NTP sincronizado em todos os nós.
* As portas exigidas para OpenShift e Portworx devem estar liberadas, além de saída TCP/443 para instalação, telemetry e licenciamento.

Validação sugerida:

```bash
oc get nodes
oc get clusterversion
oc get mcp
for n in $(oc get nodes -o name | awk -F/ '{print $2}'); do
  echo "==== $n ===="
  oc debug node/$n -- chroot /host uname -r
  oc debug node/$n -- chroot /host swapon --show
done
```

Resultado esperado:

* nós em `Ready`
* MCPs sem degradação
* swap sem saída
* kernel compatível

## 5. Preparação específica do FlashArray

Cada nó precisa alcançar o management IP do FlashArray, e o ambiente precisa já ter o data path correto conforme o protocolo escolhido.

Se usar iSCSI:

* os iniciadores iSCSI dos nós devem estar na mesma VLAN das portas target do FlashArray;
* se houver múltiplas NICs iSCSI, todas devem alcançar o management IP do FlashArray.

Se usar FC:

* os WWNs dos hosts devem estar corretamente zonados para as portas FC do FlashArray.

Validação sugerida para iSCSI:

```bash
iscsiadm -m discovery -t st -p <flasharray-interface-endpoint>
cat /etc/iscsi/initiatorname.iscsi
```

## 5.1 MachineConfigs obrigatórias

Para OpenShift com FlashArray, a documentação orienta usar MachineConfig para aplicar multipath e regras de udev de forma consistente em todos os nós.

No Lab POA, trate como obrigatório:

* aplicar MachineConfig de multipath
* aplicar MachineConfig de udev rules
* validar criação dos MCPs
* aguardar `Updated=True`
* confirmar reboot dos nós após aplicação

Validação:

```bash
oc get mcp
oc get machineconfig
```

Resultado esperado:

* MCPs `Updated=True`
* MCPs `Updating=False`
* MCPs `Degraded=False`

## 5.2 Pré-requisito de monitoring do OpenShift

Em versões mais novas do OpenShift, trate o `cluster-monitoring-config` como pré-requisito do cluster, e não como configuração do Portworx. Antes de instalar o Operator, confirme que o monitoring para user-defined projects já foi habilitado no OpenShift.

No Lab POA, considere obrigatório:

- `cluster-monitoring-config` já aplicado
- `openshift-user-workload-monitoring` operacional

Validação:

```bash
oc get configmap cluster-monitoring-config -n openshift-monitoring
oc get pods -n openshift-user-workload-monitoring
```

Mantenha o YAML de configuração do OpenShift fora deste runbook, por exemplo em `openshift/cluster-monitoring-config.yaml`.

## 5.3 Preparação do Pure Secret

Para integrar Portworx com FlashArray, você deve criar um `pure.json` com os endpoints de management e o API token, e então criar o secret **obrigatoriamente** com o nome `px-pure-secret` no mesmo namespace do `StorageCluster`.

Fluxo operacional:

* gerar `pure.json`
* validar endpoint do FlashArray
* validar API token
* criar secret `px-pure-secret`
* validar existência do secret antes do `StorageCluster`

Exemplo:

```bash
cat > pure.json <<'EOF'
{
  "FlashArrays": [
    {
      "MgmtEndPoint": "<flasharray-mgmt-endpoint>",
      "APIToken": "<flasharray-api-token>"
    }
  ]
}
EOF
```

```bash
oc create secret generic px-pure-secret \
  --namespace ${PX_NS} \
  --from-file=pure.json=./pure.json
```

Validação:

```bash
oc get secret px-pure-secret -A
oc describe secret px-pure-secret -n ${PX_NS}
```

Resultado esperado:

* secret encontrado
* `pure.json` presente

## 5.4 SSD local para metadata, se aplicável

Se você for usar SSD local para metadata no desenho do PX-StoreV2, valide antes:

* existência do SSD local para metadata
* device em modo RAW
* device visível no host
* device selecionado no Spec Generator

Se o desenho final usar metadata no próprio FlashArray, marque este item como `N/A`.

Validação sugerida por nó:

```bash
oc debug node/<node-name> -- chroot /host lsblk
```

## 6. Gerar a especificação no Portworx Central

No Portworx Central, o fluxo oficial é selecionar a versão, a plataforma FlashArray e a distribuição Kubernetes/OpenShift para gerar a especificação customizada.

Preencha assim:

* `Portworx Version`: 3.6.x
* `Platform`: Pure FlashArray
* `Distribution Name`: OpenShift / Kubernetes distribution correspondente

Na customização, revise obrigatoriamente:

* tipo de drive: `Create Using a Spec`
* backend store: `PX-StoreV2` (default no fluxo atual)
* SAN type: `iSCSI`, `NVMe-oF RDMA`, `NVMe-oF TCP` ou `Fibre Channel`
* `Pure FA Pod Name`, se aplicável ao desenho
* tamanho dos pool drives
* `Journal Device`: `None`, `Auto` ou `Custom`
* interface de dados e de management do Portworx
* nome/prefixo do cluster
* monitoring habilitado
* telemetry, se desejado

No Lab POA, confirme explicitamente:

* PX-StoreV2 habilitado
* tamanho dos cloud drives
* metadata device configurado
* Metadata Device validado
* metadata local ou FlashArray confirmado
* journal device conforme padrão definido
* número de storage nodes

Se estiver usando múltiplas NICs para iSCSI host, adicione `PURE_ISCSI_ALLOWED_IFACES` na spec do `StorageCluster`.

## 7. Instalar o Portworx Operator

Depois de gerar a spec, instale primeiro o Operator e só depois o `StorageCluster`.

No OpenShift, você pode instalar o Operator pela UI em OperatorHub, no namespace desejado.

Validação sugerida:

```bash
oc get pods -n ${PX_OPERATOR_NS}
oc get deploy -n ${PX_OPERATOR_NS}
oc get csv -A | grep -i portworx
```

Resultado esperado:

* Operator implantado
* deployment do operator disponível

## 8. Criar o StorageCluster

Com o Operator pronto e o `px-pure-secret` já criado, aplique o `StorageCluster` gerado pelo Portworx Central.

Exemplo estrutural:

```bash
oc apply -f '<url-generated-from-portworx-central-spec-gen>'
```

Validação inicial:

```bash
oc get storagecluster -A
oc describe storagecluster -n ${PX_NS} ${PX_CLUSTER_NAME}
```

Resultado esperado:

* `StorageCluster` criado
* Operator reconciliando o cluster sem erro crítico

## 9. Validação real do Portworx

Depois do deploy, valide o Portworx de verdade, não apenas o CRD.

Comandos operacionais esperados no Lab POA:

```bash
oc get pods -n ${PX_NS} -o wide
oc get storagecluster -n ${PX_NS}
PX_POD=$(oc get pod -n ${PX_NS} -l name=portworx -o jsonpath='{.items[0].metadata.name}')
oc exec -n ${PX_NS} -it ${PX_POD} -c portworx -- /opt/pwx/bin/pxctl status
oc exec -n ${PX_NS} -it ${PX_POD} -c portworx -- /opt/pwx/bin/pxctl cluster list
oc exec -n ${PX_NS} -it ${PX_POD} -c portworx -- /opt/pwx/bin/pxctl service pool show
```

Resultado esperado:

* cluster online
* todos os nós online
* pools online

A documentação também afirma que, uma vez implantado, o Portworx detecta a presença do secret do FlashArray e passa a usar o array como storage provider.

## 10. StorageClasses do ambiente

Depois da saúde do cluster estar confirmada, aplique as StorageClasses do ambiente.

No Lab POA, tratar como padrão mínimo:

* `px-app-repl1`
* `px-app-repl2`
* `px-app-repl3`
* `px-vm-block-repl1`
* `px-vm-block-repl2`
* `px-rwx-file-kubevirt`
* `px-cdi-scratch`

Aplicação sugerida:

```bash
oc apply -f storageclasses/
oc get sc | grep ^px-
```

## 11. Teste funcional com PVC

Depois de aplicar as StorageClasses, valide o provisionamento com uma PVC simples.

Exemplo:

```bash
cat <<'EOF' | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: px-test-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: px-app-repl1
  resources:
    requests:
      storage: 10Gi
EOF
```

Validação:

```bash
oc get pvc px-test-pvc
oc get pv | grep px-test-pvc
```

Resultado esperado:

* PVC `Bound`
* PV provisionado pelo Portworx

## 12. Validações finais do Lab POA

Considere este runbook concluído quando estes itens estiverem OK:

* MachineConfigs aplicadas e MCPs saudáveis
* `px-pure-secret` existente no namespace do `StorageCluster`
* Operator saudável
* `StorageCluster` criado com sucesso
* `pxctl status` mostrando cluster online
* `pxctl cluster list` com todos os nós online
* `pxctl service pool show` com pools online
* StorageClasses aplicadas
* PVC de teste em `Bound`
* Cluster pronto para PX-Central
* Cluster pronto para PX-Backup
* Cluster pronto para workloads persistentes com FlashArray


---

## Sources

- [Install Portworx Enterprise | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/install)
- [Installation of Portworx with FlashArray using Portworx Central | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/install/pure-storage/flasharray/install-flasharray)
- [System Requirements | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/prerequisites)
- [System Requirements | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/prerequisites)
- [Install Portworx with Pure Storage FlashArray - Portworx Documentation](https://docs.portworx.com/portworx-enterprise/platform/openshift/ocp-flasharray/install-flasharray/install-flasharray-cd)
- [Prepare your Environment for Portworx Installation with FlashArray | Portworx Enterprise Documentation](https://docs.portworx.com/portworx-enterprise/platform/install/pure-storage/flasharray/prepare)
- [Install Portworx with Pure Storage FlashArray with PX-StoreV2 - Portworx Documentation](https://docs.portworx.com/portworx-enterprise/platform/openshift/ocp-flasharray/install-flasharray/install-with-px-storev2)
- [Use FlashArray as Backend Storage | Portworx CSI Documentation](https://docs.portworx.com/portworx-csi/install/prepare/flash-array)