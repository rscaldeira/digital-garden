---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/metallb/runbook_metallb_openshift_4_21_15/","dg-note-properties":{}}
---

# Runbook de instalação e configuração do MetalLB no OpenShift 4.21.15

## 1. Escopo deste runbook

Este runbook foi montado para OpenShift 4.21.15, com instalação do MetalLB via Operator, namespace padrão `metallb-system`, configuração inicial em modo Layer2 e objetivo de expor Services `LoadBalancer` para workloads do Lab POA, incluindo PX-Central e PX-Backup.

O fluxo abaixo assume ambiente bare metal ou on-prem, com um range de IPs dedicado ao MetalLB e conectividade L2 válida entre os nós do cluster e a rede onde os IPs serão anunciados.

## 2. Informações do ambiente

Preencha estes dados antes de começar:

```text
Namespace: metallb-system
Pool Name: __________________
Range MetalLB: __________________
Gateway: __________________
VLAN: __________________
Subnet: __________________
Modo:
( X ) Layer2
(   ) BGP
```

## 3. Variáveis que vale definir antes de começar

Preencha estes valores antes da execução:

```bash
export METALLB_NS="metallb-system"
export METALLB_POOL_NAME="lab-poa-pool"
export METALLB_RANGE="192.168.1.240-192.168.1.250"
export METALLB_L2ADV_NAME="lab-poa-l2"
```

Você pode definir desde já namespace, nome do pool, range e nome do `L2Advertisement`. Para o Lab POA, o padrão recomendado neste runbook é manter tudo em `metallb-system` para evitar ambiguidade operacional.

## 4. Pré-check do cluster

Valide estes itens antes da instalação:

* acesso `cluster-admin` ao OpenShift;
* `oc` instalado e funcional;
* catálogo com `metallb-operator` disponível;
* cluster sem uso de IP failover, porque IP failover e MetalLB são incompatíveis;
* range de IPs previamente reservado para o MetalLB;
* confirmação de que a rede física entrega tráfego para esses IPs até os nós do cluster.

Validação sugerida:

```bash
oc whoami
oc get clusterversion
oc get nodes
oc get packagemanifests -n openshift-marketplace metallb-operator
```

## 5. Preparação do range de IPs

Antes de instalar, valide o range que será usado pelo MetalLB.

Critérios práticos:

* o range não pode estar sendo usado por DHCP;
* o range não deve colidir com VIPs existentes;
* o range deve estar na subnet correta do ambiente;
* a rede deve permitir que um nó anuncie esse IP via Layer2.

Para o Lab POA, documente no runbook local:

```text
Range escolhido:
Gateway:
Subnet:
VLAN:
Hosts já reservados:
```

## 6. Instalação do Operator

Este runbook usa o caminho por CLI com namespace fixo `metallb-system`.

### 6.1. Criar o namespace

```bash
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
EOF
```

### 6.2. Habilitar cluster monitoring no namespace

Opcional, mas recomendado se você quiser visibilidade de métricas do MetalLB no cluster monitoring.

```bash
oc label ns ${METALLB_NS} "openshift.io/cluster-monitoring=true" --overwrite
```

### 6.3. Criar o OperatorGroup

```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: metallb-operator
  namespace: ${METALLB_NS}
spec:
  targetNamespaces:
  * ${METALLB_NS}
EOF
```

### 6.4. Criar a Subscription

```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator-sub
  namespace: ${METALLB_NS}
spec:
  channel: stable
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

### 6.5. Validar InstallPlan e CSV

```bash
oc get installplan -n ${METALLB_NS}
oc get csv -n ${METALLB_NS}
oc get clusterserviceversion -n ${METALLB_NS} \
  -o custom-columns=Name:.metadata.name,Phase:.status.phase
```

Resultado esperado:

* Operator instalado
* CSV em `Succeeded`

### 6.6. Validar os CRDs do MetalLB

Essa validação não é obrigatória, mas ajuda bastante quando o Operator instala parcialmente.

```bash
oc get crd | grep metallb
```

Resultado esperado:

* CRDs do MetalLB presentes

## 7. Ativar o MetalLB no cluster

Depois que o Operator estiver pronto, crie a instância única do recurso `MetalLB`. Deve existir apenas uma por cluster.

```bash
cat << EOF | oc apply -f -
apiVersion: metallb.io/v1beta1
kind: MetalLB
metadata:
  name: metallb
  namespace: ${METALLB_NS}
EOF
```

Validação sugerida:

```bash
oc get metallb -A
```

Resultado esperado:

* recurso `MetalLB` presente
* `MetalLB` em estado Ready

## 8. Validar o deploy do MetalLB

Valide os componentes principais:

```bash
oc get deployment -n ${METALLB_NS} controller
oc get daemonset -n ${METALLB_NS} speaker
oc get all -n ${METALLB_NS}
```

Resultado esperado:

* `controller` disponível
* `speaker` rodando em todos os nós elegíveis do cluster

## 9. Configuração inicial em Layer2

Para Layer2, o MetalLB precisa de pelo menos:

* um `IPAddressPool`
* um `L2Advertisement` associado ao pool

Antes de criar o pool, valide se nenhum IP da faixa escolhida já responde na rede. Isso ajuda a evitar conflitos estranhos com IPs já em uso.

Exemplo de validação:

```bash
for ip in $(seq 240 250); do ping -c 1 -W 1 192.168.1.$ip >/dev/null && echo "IP em uso: 192.168.1.$ip"; done
```

Resultado esperado:

* nenhum IP do range responde a ping antes da criação do `IPAddressPool`

### 9.1. Criar o pool e o anúncio L2

```bash
cat << EOF | oc apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ${METALLB_POOL_NAME}
  namespace: ${METALLB_NS}
spec:
  addresses:
  * ${METALLB_RANGE}
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: ${METALLB_L2ADV_NAME}
  namespace: ${METALLB_NS}
spec:
  ipAddressPools:
  * ${METALLB_POOL_NAME}
EOF
```

### 9.2. Validar os CRs

```bash
oc get ipaddresspool -A
oc get l2advertisement -A
```

Resultado esperado:

* `IPAddressPool` criado
* `L2Advertisement` criado

## 10. Ajustes opcionais de Layer2

Se quiser restringir quem anuncia o IP:

* usar `nodeSelectors` no `L2Advertisement`
* usar `interfaces` no `L2Advertisement`

Exemplo de uso quando houver necessidade:

```yaml
spec:
  ipAddressPools:
  * lab-poa-pool
  nodeSelectors:
  - matchLabels:
      node-role.kubernetes.io/worker: ""
  interfaces:
  - ens192
```

Atenção: se você restringir interfaces ou nós de forma incorreta, o MetalLB pode eleger um nó incapaz de anunciar o IP corretamente.

## 11. Teste funcional obrigatório com Service real

Este teste é obrigatório para validar o ambiente.

### 11.1. Criar deployment de teste

```bash
oc create deployment nginx --image=nginx
```

### 11.2. Expor o deployment internamente

```bash
oc expose deployment nginx --port=80
```

### 11.3. Criar um Service LoadBalancer

```bash
oc expose svc nginx \
  --name nginx-lb \
  --type LoadBalancer
```

### 11.4. Validar a alocação do IP

```bash
oc get svc nginx-lb
```

Resultado esperado:

* `EXTERNAL-IP` atribuído
* IP vindo do range definido em `${METALLB_RANGE}`

### 11.5. Validar acesso ao IP

De um host da mesma rede, valide conectividade para o IP recebido:

```bash
curl http://<external-ip>
```

Se necessário para troubleshooting L2:

```bash
arping <external-ip>
```

## 12. Troubleshooting básico

Se o `EXTERNAL-IP` não vier ou o serviço não responder:

### 12.1. Verificar os CRs

```bash
oc get metallb -A
oc get ipaddresspool -A
oc get l2advertisement -A
```

### 12.2. Verificar pods e eventos

```bash
oc get pods -n ${METALLB_NS}
oc describe svc nginx-lb
oc get events -n ${METALLB_NS} --sort-by='.lastTimestamp'
```

### 12.3. Verificar logs

```bash
oc logs -n ${METALLB_NS} deployment/controller
oc logs -n ${METALLB_NS} daemonset/speaker
```

### 12.4. Verificar se a configuração ficou stale

O MetalLB pode marcar configuração inválida como stale e continuar com a última configuração válida.

Se você tiver Prometheus, observe a métrica:

```text
metallb_k8s_client_config_stale_bool
```

## 13. Limpeza do teste

Depois do teste funcional, se quiser limpar:

```bash
oc delete svc nginx-lb
oc delete svc nginx
oc delete deployment nginx
```

## 14. Critérios de aceite

Considere este runbook concluído quando estes itens estiverem OK:

* Operator `metallb-operator` instalado com status `Succeeded`.
* recurso `MetalLB` criado com sucesso.
* `controller` disponível.
* `speaker` rodando em todos os nós elegíveis.
* pelo menos um `IPAddressPool` criado.
* pelo menos um `L2Advertisement` criado.
* serviço de teste `nginx-lb` recebendo `EXTERNAL-IP` do range planejado
* tráfego chegando corretamente ao serviço exposto

## 15. Validação para Lab POA

* [ ] PX-Central poderá usar `Service LoadBalancer`
* [ ] PX-Backup poderá usar `Service LoadBalancer`
* [ ] Cluster pronto para workloads que dependem de `LoadBalancer`
* [ ] Cluster pronto para PX-Central
* [ ] Cluster pronto para PX-Backup


---

## Sources

- [Installing the MetalLB Operator - Networking Operators](https://docs.okd.io/4.18/networking/networking_operators/metallb-operator/metallb-operator-install.html)
- [Configuration :: MetalLB, bare metal load-balancer for Kubernetes](https://metallb.io/configuration/)
- [Chapter 23. Load balancing with MetalLB](https://docs.redhat.com/en/documentation/openshift_container_platform/4.9/html/networking/load-balancing-with-metallb)
- [Chapter 5. MetalLB Operator](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/networking_operators/metallb-operator)
- [GitHub - metallb/metallb-operator: MetalLB Operator for deploying ...](https://github.com/metallb/metallb-operator)
- [Troubleshooting MetalLB :: MetalLB, bare metal load-balancer for ...](https://metallb.io/troubleshooting/)
- [Advanced L2 configuration :: MetalLB, bare metal load-balancer ...](https://metallb.io/configuration/_advanced_l2_configuration/)
- [API reference docs :: MetalLB, bare metal load-balancer for Kubernetes](https://metallb.io/apis/)