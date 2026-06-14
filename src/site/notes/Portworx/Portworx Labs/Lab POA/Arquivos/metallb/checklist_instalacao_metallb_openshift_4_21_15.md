---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/metallb/checklist_instalacao_metallb_openshift_4_21_15/","dg-note-properties":{}}
---

# Checklist de instalação e configuração do MetalLB no OpenShift 4.21.15

## Escopo

* [ ] Este checklist assume OpenShift 4.21.15 com instalação do MetalLB via Operator e configuração inicial em modo Layer 2, que é o modo mais simples de configurar no MetalLB.
* [ ] Confirmar que o objetivo é expor Services do tipo `LoadBalancer` em ambiente bare metal/on-prem com pool de IPs dedicado ao MetalLB.

## Informações do Ambiente

- Namespace: `metallb-system`
- Pool Name:
- Range MetalLB:
- Gateway:
- VLAN:
- Subnet:
- Modo:
  - ( X ) Layer2
  - (   ) BGP

## 1. Pré-requisitos do cluster

* [ ] Confirmar acesso `cluster-admin` ao cluster OpenShift.
* [ ] Confirmar que o `oc` está instalado e funcional.
* [ ] Confirmar que o Operator `metallb-operator` está disponível no catálogo do cluster.

```bash
oc get packagemanifests -n openshift-marketplace metallb-operator
```

* [ ] Validar se o cluster usa IP failover. MetalLB e IP failover são incompatíveis; se IP failover estiver configurado, ele deve ser removido antes da instalação.
* [ ] Definir previamente o range de IPs que será entregue pelo MetalLB aos Services `LoadBalancer`.
* [ ] Confirmar que a rede do ambiente roteia ou entrega tráfego para esses IPs até os nós do cluster.

## 2. Namespace e método de instalação

* [ ] Definir se a instalação do Operator será pela web console ou por CLI.
* [ ] Para o Lab POA, padronizar o namespace do MetalLB como `metallb-system`.
* [ ] Manter consistência entre Operator, `MetalLB`, `IPAddressPool` e `L2Advertisement`, usando `metallb-system` como namespace operacional.

## 3. Instalação do Operator

### Caminho A: OperatorHub pela web console

* [ ] Acessar `Operators` → `OperatorHub`.
* [ ] Procurar por `metallb`.
* [ ] Selecionar `MetalLB Operator`.
* [ ] Instalar aceitando os defaults do ambiente, se isso estiver alinhado ao padrão do cluster.
* [ ] Validar em `Operators` → `Installed Operators` que o status ficou `Succeeded`.

### Caminho B: Instalação por CLI

* [ ] Criar o namespace `metallb-system`.

```bash
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
EOF
```

* [ ] Opcionalmente, habilitar métricas do namespace para cluster monitoring, se quiser visibilidade de BGP/BFD no Prometheus do cluster.

```bash
oc label ns metallb-system "openshift.io/cluster-monitoring=true"
```

* [ ] Criar o `OperatorGroup` no namespace.

```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: metallb-operator
  namespace: metallb-system
spec:
  targetNamespaces:
  * metallb-system
EOF
```

* [ ] Criar a `Subscription` usando `redhat-operators` como source.

```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator-sub
  namespace: metallb-system
spec:
  channel: stable
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

* [ ] Confirmar o `InstallPlan`.

```bash
oc get installplan -n metallb-system
```

* [ ] Confirmar que o CSV do Operator chegou a `Succeeded`.

```bash
oc get clusterserviceversion -n metallb-system \
  -o custom-columns=Name:.metadata.name,Phase:.status.phase
```

## 4. Ativar o MetalLB no cluster

* [ ] Depois que o Operator estiver instalado, criar uma única instância do recurso `MetalLB`; é isso que efetivamente inicia o MetalLB no cluster.
* [ ] Garantir que exista somente um recurso `MetalLB` por cluster.

```bash
cat << EOF | oc apply -f -
apiVersion: metallb.io/v1beta1
kind: MetalLB
metadata:
  name: metallb
  namespace: metallb-system
EOF
```

* [ ] Para o Lab POA, manter o recurso `MetalLB` em `metallb-system` e usar esse mesmo namespace para os CRs de configuração.

## 5. Validar o deploy do MetalLB

* [ ] Confirmar que o deployment `controller` está `Available`.

```bash
oc get deployment -n metallb-system controller
```

* [ ] Confirmar que o daemonset `speaker` está rodando.

```bash
oc get daemonset -n metallb-system speaker
```

* [ ] Validar que existe um `speaker` por nó elegível do cluster.
* [ ] Se necessário, listar todos os recursos do namespace para troubleshooting inicial.

```bash
oc get all -n metallb-system
```

## 6. Configuração inicial em Layer 2

* [ ] Criar pelo menos um `IPAddressPool` com o range de IPs que o MetalLB poderá alocar aos Services `LoadBalancer`.
* [ ] Criar um `L2Advertisement` associado ao pool, porque no modo Layer 2 o MetalLB só anuncia IPs de um pool quando existe um `L2Advertisement` correspondente.
* [ ] Confirmar que o range escolhido não está em uso por DHCP, roteadores, VIPs existentes ou outros serviços do ambiente.

Exemplo base:

```bash
cat << EOF | oc apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  * 192.168.1.240-192.168.1.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  * first-pool
EOF
```

* [ ] Entender a regra do `L2Advertisement`: se você não informar seleção de pool, ele pode se aplicar a todos os `IPAddressPools`; se houver pools especializados, declare explicitamente os pools que quer anunciar via L2.
* [ ] Validar a criação dos CRs de configuração.

```bash
oc get ipaddresspool -A
oc get l2advertisement -A
```

Resultado esperado:
* `IPAddressPool` criado
* `L2Advertisement` criado

## 7. Restrições e ajustes opcionais de L2

* [ ] Se quiser restringir quais nós podem anunciar determinados IPs, usar `nodeSelectors` no `L2Advertisement`.
* [ ] Se quiser restringir por interface física, usar `interfaces` no `L2Advertisement`.
* [ ] Validar bem essas restrições antes de aplicar, porque o MetalLB pode eleger um nó sem a interface esperada e o serviço deixar de ser anunciado corretamente.

## 8. Configuração BGP opcional

* [ ] Se o ambiente usar BGP em vez de L2, criar `BGPPeer` e `BGPAdvertisement` em vez de depender apenas de `L2Advertisement`.
* [ ] Preferir `BGPPeer` em `v1beta2`, porque `v1beta1` está marcado como deprecated.
* [ ] Ter previamente definidos: IP do roteador, ASN do peer, ASN do MetalLB e range/CIDR a anunciar.

## 9. Validação funcional

### Service de teste

* [ ] Criar um deployment de teste com `nginx`.

```bash
oc create deployment nginx --image=nginx
```

* [ ] Expor o deployment internamente.

```bash
oc expose deployment nginx --port=80
```

* [ ] Criar um Service `LoadBalancer` para o teste.

```bash
oc expose svc nginx \
  --name nginx-lb \
  --type LoadBalancer
```

* [ ] Validar a criação do serviço.

```bash
oc get svc nginx-lb
```

Resultado esperado:
* `EXTERNAL-IP` atribuído

* [ ] Validar conectividade até esse IP a partir de um cliente da rede correta.
* [ ] Em L2, se necessário para troubleshooting, usar `arping` a partir de um host na mesma sub-rede para verificar qual MAC está anunciando o IP do LoadBalancer.

## 10. Validação de saúde da configuração

* [ ] Se um serviço não receber IP ou não ficar acessível, revisar logs e eventos do `controller` e do `speaker`.
* [ ] Entender que a configuração do MetalLB é a composição de múltiplos CRs, como `IPAddressPool`, `L2Advertisement` e `BGPAdvertisement`; uma configuração inválida pode ser marcada como stale e o MetalLB continuar usando a última configuração válida.
* [ ] Se houver Prometheus, observar a métrica `metallb_k8s_client_config_stale_bool` para identificar configuração obsoleta/inválida.

## 11. Critérios de aceite

* [ ] Operator `metallb-operator` instalado com status `Succeeded`.
* [ ] Recurso `MetalLB` criado com sucesso.
* [ ] `controller` disponível.
* [ ] `speaker` rodando em todos os nós elegíveis.
* [ ] Pelo menos um `IPAddressPool` criado.
* [ ] Pelo menos um `L2Advertisement` criado para o pool usado em Layer 2.
* [ ] Service de teste `LoadBalancer` recebendo `EXTERNAL-IP` do range planejado.
* [ ] Tráfego chegando corretamente ao serviço exposto.
* [ ] Cluster pronto para expor workloads que dependem de `LoadBalancer`.

### Validação para Lab POA

* [ ] PX-Central poderá usar Service `LoadBalancer`.
* [ ] PX-Backup poderá usar Service `LoadBalancer`.
* [ ] Cluster pronto para workloads que dependem de `LoadBalancer`.


---

## Sources

- [Configuration :: MetalLB, bare metal load-balancer for Kubernetes](https://metallb.io/configuration/)
- [Chapter 23. Load balancing with MetalLB](https://docs.redhat.com/en/documentation/openshift_container_platform/4.9/html/networking/load-balancing-with-metallb)
- [Installing the MetalLB Operator - Networking Operators](https://docs.okd.io/4.18/networking/networking_operators/metallb-operator/metallb-operator-install.html)
- [Chapter 5. MetalLB Operator](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/networking_operators/metallb-operator)
- [GitHub - metallb/metallb-operator: MetalLB Operator for deploying ...](https://github.com/metallb/metallb-operator)
- [Troubleshooting MetalLB :: MetalLB, bare metal load-balancer for ...](https://metallb.io/troubleshooting/)
- [Advanced L2 configuration :: MetalLB, bare metal load-balancer ...](https://metallb.io/configuration/_advanced_l2_configuration/)
- [API reference docs :: MetalLB, bare metal load-balancer for Kubernetes](https://metallb.io/apis/)