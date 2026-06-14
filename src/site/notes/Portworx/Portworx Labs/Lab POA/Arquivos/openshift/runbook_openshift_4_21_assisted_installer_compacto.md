---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/openshift/runbook_openshift_4_21_assisted_installer_compacto/","dg-note-properties":{}}
---

# Runbook de instalação do OpenShift 4.21 com Assisted Installer

## 1. Objetivo

Este runbook descreve a instalação de um cluster OpenShift Container Platform 4.21 on-premises usando o Assisted Installer no Red Hat Hybrid Cloud Console, no cenário de cluster compacto com 3 nós, sem workers dedicados, com discovery dos hosts via DHCP e execução do fluxo somente pela interface web. O Assisted Installer é a solução guiada da Red Hat para instalação on-prem, com foco em reduzir a complexidade e a intervenção manual durante o deploy.

## 2. Premissas deste cenário

Este documento assume o seguinte desenho operacional:

* Cluster compacto com 3 nós.
* Os 3 nós serão usados como control plane.
* O discovery inicial dos hosts será feito por DHCP.
* A instalação será conduzida somente pela interface web do Assisted Installer.
* A rede do cluster será cluster-managed networking.

## 3. Pré-requisitos obrigatórios

Antes de abrir o Assisted Installer, tenha em mãos os dados mínimos abaixo:

* Nome do cluster.
* Base domain.
* Pull secret do OpenShift.
* Chave pública SSH, se você quiser acesso SSH aos nós para troubleshooting.
* Três hosts que serão inicializados com a discovery ISO.

O Assisted Installer usa o pull secret como entrada da definição do cluster, e a chave pública SSH pode ser fornecida para permitir acesso remoto aos nós durante troubleshooting e validação.

Se você optar por discovery via DHCP, garanta que o endereçamento e as rotas sejam entregues dinamicamente pela rede. O Assisted Installer não espera rotas estáticas quando o host recebeu IP dinâmico; misturar IP dinâmico com rota estática pode fazer a validação de rede falhar.

Se o hostname não vier por DHCP nem por outra fonte equivalente, prepare PTR para os nós; caso contrário, o Assisted Installer pode renomear os hosts para o MAC address da interface de rede.

Para multi-node com cluster-managed networking, o Assisted Installer não exige DNS externo para concluir a instalação. Esses registros podem ser publicados depois, quando você quiser acessar API e ingress a partir de fontes externas ao cluster.

## 4. Planejamento de rede para este cenário

Para a configuração final do cluster, planeje os seguintes itens antes de começar:

* machine network CIDR
* cluster network CIDR
* service network CIDR
* API VIP
* Ingress VIP
* resolução DNS final do cluster para acesso externo pós-instalação

Para OpenShift 4.21, use `OVNKubernetes` como tipo de rede neste cenário. Nas validações do Assisted Installer, `OVNKubernetes` é aceito e o recurso de VIP DHCP allocation não é suportado com `OVNKubernetes`, então não planeje depender de autoalocação de API VIP e Ingress VIP por DHCP.

Na prática, isso significa:

* DHCP pode ser usado para o discovery dos hosts.
* A configuração final do cluster deve ter API VIP e Ingress VIP definidos manualmente.
* Esses VIPs devem pertencer à machine network e não podem estar em uso.

Como referência de valores típicos mostrados pela documentação do Assisted Installer para cluster IPv4 com OVNKubernetes, os exemplos usam:

* cluster network: `10.128.0.0/14`
* host prefix: `23`
* service network: `172.30.0.0/16`
* machine network: a sua sub-rede real do ambiente

## 5. DNS e portas para acesso externo

Quando você for expor o cluster para acesso externo após a instalação, publique pelo menos os seguintes registros DNS:

* `api.<cluster>.<base_domain>`
* `api-int.<cluster>.<base_domain>`
* `*.apps.<cluster>.<base_domain>`

Não crie wildcard genérico fora do padrão esperado do cluster, porque a documentação alerta que um wildcard inadequado pode impedir o prosseguimento da instalação.

Se houver firewall entre administradores, usuários e o cluster, considere no mínimo:

* porta 6443 para acesso à API com `oc`
* porta 443 para console e tráfego de ingress
* porta 22623 para o Machine Config Server servir ignition via TLS
* porta 22624 para cenários de compatibilidade e provisionamento relacionados a novos nós

## 6. Passo a passo de instalação

### Etapa 1 – Abrir o Assisted Installer

Acesse o Red Hat Hybrid Cloud Console e abra o Assisted Installer. Esse é o ponto de entrada oficial do fluxo web para instalação on-prem com baixa intervenção manual.

### Etapa 2 – Criar o cluster

Crie um novo cluster e preencha:

* nome do cluster
* base domain
* versão `4.21`
* topologia com `3` control planes

No modelo do Assisted Installer, um cluster multi-node pode ser definido com `3`, `4` ou `5` control planes; para este runbook, use `3` porque o objetivo é cluster compacto.

### Etapa 3 – Configurar a rede do cluster

Na tela de networking, configure:

* `networkType = OVNKubernetes`
* `machineNetwork` com o CIDR da sub-rede real dos hosts
* `clusterNetwork`
* `serviceNetwork`
* `apiVIP`
* `ingressVIP`

Para este cenário, use DHCP apenas para o discovery dos hosts. Não habilite uma estratégia dependente de VIP DHCP allocation para o cluster final, porque o Assisted Installer valida `OVNKubernetes` sem suporte a esse mecanismo.

### Etapa 4 – Adicionar pull secret e chave SSH

No fluxo do cluster, informe o pull secret. Adicione também a chave pública SSH se quiser acesso aos hosts para debug ou suporte operacional.

### Etapa 5 – Gerar a discovery ISO

Gere a discovery ISO para registrar os hosts no Assisted Installer. A discovery ISO é o artefato que inicializa o agente de discovery e prepara o host para validação e instalação.

Se você for inicializar por mídia virtual com largura de banda limitada, prefira a minimal ISO. Se não houver essa restrição, use a ISO completa para simplificar o boot do host.

### Etapa 6 – Inicializar os 3 hosts com a discovery ISO

Anexe a discovery ISO a cada um dos 3 hosts e ajuste a ordem de boot para inicializar por essa mídia. Depois, volte ao navegador e aguarde os hosts aparecerem na lista de discovered hosts.

Se algum host não aparecer:

* confirme que ele realmente iniciou pela discovery ISO
* confirme que o DHCP está funcional
* confirme que a rede do host alcança o serviço do Assisted Installer

### Etapa 7 – Validar os hosts descobertos

Na lista de hosts, confirme que os 3 nós ficaram em estado pronto para instalação. O Assisted Installer só libera o início quando todos os hosts estiverem prontos e quando a contagem de masters atender ao número definido para o cluster.

Como este é um cluster compacto, mantenha os 3 hosts como control plane.

### Etapa 8 – Iniciar a instalação

Com os 3 hosts em estado pronto, inicie a instalação pelo Assisted Installer. A própria plataforma executa as validações de pré-instalação e bloqueia o avanço quando algum requisito essencial não está atendido.

### Etapa 9 – Acompanhar o progresso

Acompanhe o progresso pelo console web até a conclusão. Caso haja falha, use o acesso por console/BMC ou SSH no host para verificar rede e boot do discovery agent.

Para teste rápido de acesso SSH ao host, use o formato abaixo:

```bash
ssh -i <ssh_private_key_file> core@<host_ip_address>
```

### Etapa 10 – Coletar os dados de acesso ao cluster

Quando a instalação terminar, o Assisted Installer informa que o processo foi concluído e disponibiliza os dados principais de acesso, incluindo a URL do console web e as credenciais do usuário `kubeadmin`.

## 7. Pós-instalação imediata

Depois que o cluster subir:

* publique os registros DNS externos finais para API e apps, se ainda não tiver feito isso
* valide acesso ao console
* valide acesso à API
* registre os nomes definitivos dos nós conforme sua convenção operacional

Como o seu cenário usa cluster-managed networking, a publicação desses registros DNS externos pode ser feita após a instalação para habilitar o acesso externo ao cluster.

## 8. Critérios de aceite

Considere a instalação concluída quando estes pontos estiverem atendidos:

* Os 3 hosts apareceram no Assisted Installer e chegaram ao estado pronto.
* A instalação terminou sem bloqueios.
* O console do OpenShift abre pela URL final.
* O login com `kubeadmin` funciona.
* API VIP e Ingress VIP respondem conforme o desenho da rede.
* Os registros DNS externos finais foram publicados para o consumo do cluster.

## 9. Pontos de atenção deste cenário

* DHCP para discovery não elimina a necessidade de planejar manualmente `apiVIP` e `ingressVIP` no OpenShift 4.21 com `OVNKubernetes`.
* Se os hostnames não vierem corretamente da rede, os nós podem aparecer com nomes derivados do MAC address.
* Se houver firewall, trate desde o início as portas 6443, 443, 22623 e 22624 para não travar validação, configuração ou acesso ao cluster.


---

## Sources

- [Chapter 1. Installing an on-premise cluster using the Assisted Installer | Installing on-premise with Assisted Installer | OpenShift Container Platform | 4.21 | Red Hat Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/installing_on-premise_with_assisted_installer/installing-on-prem-assisted)
- [OpenShift Container Platform 4.21 Installing an on-premise cluster ...](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/pdf/installing_an_on-premise_cluster_with_the_agent-based_installer/OpenShift_Container_Platform-4.21-Installing_an_on-premise_cluster_with_the_Agent-based_Installer-en-US.pdf)
- [Chapter 5. Installing with the Assisted Installer API](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2025/html/installing_openshift_container_platform_with_the_assisted_installer/installing-with-api)
- [OpenShift Container Platform 4.21 Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/pdf/installing_on_a_single_node/OpenShift_Container_Platform-4.21-Installing_on_a_single_node-en-US.pdf)
- [Chapter 2. Prerequisites](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2026/html/installing_openshift_container_platform_with_the_assisted_installer/prerequisites)
- [Installing OpenShift Container Platform with the Assisted Installer ...](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2026/html-single/installing_openshift_container_platform_with_the_assisted_installer/index)
- [Chapter 2. Prerequisites](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2026/html/installing_openshift_container_platform_with_the_assisted_installer/prerequisites)
- [Chapter 11. Network configuration](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2023/html/assisted_installer_for_openshift_container_platform/assembly_network-configuration)
- [Chapter 10. Network configuration](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2026/html/installing_openshift_container_platform_with_the_assisted_installer/assembly_network-configuration)
- [Chapter 2. Prerequisites](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2024/html/installing_openshift_container_platform_with_the_assisted_installer/prerequisites)
- [Chapter 12. Installing on-premise with Assisted Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/4.11/html/installing/installing-on-premise-with-assisted-installer)
- [Assisted Installer for OpenShift Container Platform 2026](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2025/pdf/installing_openshift_container_platform_with_the_assisted_installer/Assisted_Installer_for_OpenShift_Container_Platform-2025-Installing_OpenShift_Container_Platform_with_the_Assisted_Installer-en-US.pdf)
- [Chapter 8. Booting hosts with the discovery image](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2026/html/installing_openshift_container_platform_with_the_assisted_installer/assembly_booting-hosts-with-the-discovery-image)
- [Chapter 15. Troubleshooting](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2026/html/installing_openshift_container_platform_with_the_assisted_installer/assembly_troubleshooting)
- [Chapter 15. Troubleshooting](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2023/html/assisted_installer_for_openshift_container_platform/assembly_troubleshooting)
- [Chapter 11. Installing on-premise with Assisted Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/4.10/html/installing/installing-on-premise-with-assisted-installer)
- [Chapter 15. Troubleshooting](https://docs.redhat.com/en/documentation/assisted_installer_for_openshift_container_platform/2026/html/installing_openshift_container_platform_with_the_assisted_installer/assembly_troubleshooting)
- [OpenShift Container Platform 4.21 Installing on Oracle Database ...](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/pdf/installing_on_oracle_database_appliance/OpenShift_Container_Platform-4.21-Installing_on_Oracle_Database_Appliance-en-US.pdf)
