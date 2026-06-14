---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/openshift/checklist_instalacao_openshift_4_21_assisted_installer/","dg-note-properties":{}}
---

# Checklist de instalação e configuração do OpenShift 4.21 com Assisted Installer

## Escopo

* [ ] Este checklist assume o mesmo cenário do runbook: cluster compacto com 3 nós, sem workers dedicados, discovery inicial via DHCP e instalação 100% pela interface web do Assisted Installer.
* [ ] Confirmar que o método será o Assisted Installer para hardware on-prem ou VMs on-prem, via Hybrid Cloud Console.
* [ ] Confirmar que o time quer seguir o fluxo guiado com baixa intervenção manual.

## 1. Definições iniciais

* [ ] Definir o nome do cluster.
* [ ] Definir o base domain.
* [ ] Confirmar que a versão alvo é OpenShift 4.21.
* [ ] Confirmar a topologia compacta com 3 control planes.
* [ ] Confirmar que não haverá workers dedicados neste primeiro deploy.

## 2. Inventário e hosts

* [ ] Separar 3 hosts para o cluster.
* [ ] Confirmar CPU, memória, disco e NICs de cada host conforme o padrão interno do ambiente.
* [ ] Confirmar acesso remoto ao BMC/iDRAC/iLO/console virtual de cada host.
* [ ] Confirmar ordem de boot pela mídia ISO.
* [ ] Confirmar que os hosts podem receber IP por DHCP durante o discovery.

## 3. Rede e endereçamento

* [ ] Confirmar que o DHCP está funcional na rede onde os hosts vão iniciar.
* [ ] Definir a machine network CIDR.
* [ ] Definir a cluster network CIDR.
* [ ] Definir a service network CIDR.
* [ ] Reservar o API VIP.
* [ ] Reservar o Ingress VIP.
* [ ] Confirmar que os VIPs pertencem à mesma rede esperada para o cluster.
* [ ] Confirmar gateway e DNS que os nós usarão no ambiente final.
* [ ] Definir a convenção de hostname dos 3 nós.

## 4. DNS e acesso externo

* [ ] Planejar os registros `api.<cluster>.<base_domain>`.
* [ ] Planejar os registros `api-int.<cluster>.<base_domain>`.
* [ ] Planejar o wildcard `*.apps.<cluster>.<base_domain>`.
* [ ] Confirmar quem publicará esses registros e em que momento.
* [ ] Confirmar acesso às portas administrativas exigidas no ambiente.
* [ ] Validar antecipadamente o caminho de acesso externo à API e ao console após a instalação.

## 5. Credenciais e insumos

* [ ] Baixar o pull secret do OpenShift.
* [ ] Separar a chave pública SSH que será usada nos hosts, se desejado.
* [ ] Confirmar conta com acesso ao Red Hat Hybrid Cloud Console.
* [ ] Confirmar acesso ao Assisted Installer pela interface web.

## 6. Criação do cluster no Assisted Installer

* [ ] Acessar o Assisted Installer no Hybrid Cloud Console.
* [ ] Criar um novo cluster.
* [ ] Informar nome do cluster.
* [ ] Informar base domain.
* [ ] Selecionar a versão 4.21.
* [ ] Definir a topologia com 3 control planes.
* [ ] Informar o pull secret.
* [ ] Informar a chave pública SSH, se for usar.
* [ ] Definir `OVNKubernetes` como network type, se esse for o padrão adotado no projeto.
* [ ] Preencher machine network, cluster network e service network.
* [ ] Preencher API VIP e Ingress VIP.

## 7. Discovery ISO

* [ ] Gerar a discovery ISO no Assisted Installer.
* [ ] Decidir se será usada ISO completa ou minimal.
* [ ] Anexar a ISO aos 3 hosts.
* [ ] Bootar os 3 hosts pela discovery ISO.
* [ ] Validar que os hosts aparecem no Assisted Installer.
* [ ] Confirmar que cada host foi identificado corretamente.
* [ ] Ajustar hostname, se necessário.
* [ ] Verificar se os 3 hosts ficaram em estado apto para instalação.

## 8. Validações antes de instalar

* [ ] Confirmar que não há alertas críticos pendentes no Assisted Installer.
* [ ] Confirmar que os 3 hosts estão atribuídos ao papel de control plane.
* [ ] Confirmar que a rede final está coerente com os VIPs escolhidos.
* [ ] Confirmar que os 3 hosts enxergam a rede esperada.
* [ ] Confirmar que a topologia compacta está correta antes de iniciar.
* [ ] Revisar hostname, IP, MAC e papel de cada host.

## 9. Execução da instalação

* [ ] Iniciar a instalação pelo Assisted Installer.
* [ ] Acompanhar o progresso no console web.
* [ ] Monitorar eventuais falhas de validação.
* [ ] Se necessário, acessar console/BMC/SSH dos hosts para troubleshooting.
* [ ] Aguardar a conclusão completa do cluster.

## 10. Pós-instalação imediata

* [ ] Coletar a URL do console.
* [ ] Coletar a credencial inicial do `kubeadmin`.
* [ ] Testar acesso ao console web.
* [ ] Testar acesso à API com `oc login`.
* [ ] Publicar ou validar os registros DNS externos finais, se isso ainda não tiver sido feito.
* [ ] Confirmar que API VIP e Ingress VIP respondem corretamente.
* [ ] Confirmar que os 3 nós aparecem no cluster.

## 11. Configuração inicial do cluster

* [ ] Rotacionar ou guardar com segurança a credencial inicial de acesso.
* [ ] Validar o estado dos nodes.
* [ ] Validar o estado dos cluster operators.
```bash
oc get co
```
Resultado esperado:

- `Available=True`
- `Degraded=False`
- `Progressing=False` ou apenas alguns ainda finalizando

* [ ] Validar o estado dos MachineConfigPools.
```bash
oc get mcp
```
Resultado esperado:
- `Updated=True`
- `Updating=False`
- `Degraded=False`

* [ ] Validar o estado dos pods críticos em `openshift-*`.
* [ ] Configurar usuários e grupos administrativos iniciais.
* [ ] Definir o provider de autenticação corporativo, se aplicável.
* [ ] Validar storage classes disponíveis.
* [ ] Validar ingress e certificados conforme o padrão do ambiente.
* [ ] Validar image registry, se fizer parte do escopo inicial.
* [ ] Documentar os VIPs, DNS, hosts e credenciais operacionais.

## 12. Critérios de aceite

* [ ] Os 3 hosts foram descobertos com sucesso.
* [ ] O cluster foi instalado sem erro.
* [ ] O console abre pela URL final.
* [ ] O login funciona.
* [ ] A API responde.
* [ ] Os 3 nós estão em estado Ready.
* [ ] Os operators principais estão Available.
* [ ] Os registros DNS finais estão corretos.
* [ ] API VIP e Ingress VIP estão funcionais.
* [ ] O ambiente está pronto para receber configuração pós-instalação e workloads.
* [ ] Cluster pronto para instalação do MetalLB.
* [ ] Cluster pronto para aplicação dos MachineConfigs Portworx.
* [ ] Cluster pronto para instalação do Portworx Operator.


---

## Sources

- [Chapter 1. Installing an on-premise cluster using the Assisted Installer | Installing on-premise with Assisted Installer | OpenShift Container Platform | 4.21 | Red Hat Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/installing_on-premise_with_assisted_installer/installing-on-prem-assisted)
