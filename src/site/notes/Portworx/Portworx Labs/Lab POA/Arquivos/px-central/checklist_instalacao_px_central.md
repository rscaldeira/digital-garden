---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/px-central/checklist_instalacao_px_central/","dg-note-properties":{}}
---

# Checklist de instalação e configuração do PX-Central

## Escopo e premissas

* [ ] Este checklist assume Portworx Central on-prem 2.11 em ambiente non-airgap. O produto pode ser instalado em um cluster Kubernetes já com Portworx ou em um cluster Kubernetes novo sem Portworx.
* [ ] Definir antes do início se o ambiente terá somente PX-Central, ou PX-Central com monitoring, com license server, e com OIDC interno ou externo.
* [ ] Considerar a arquitetura base: application gateway, servidor OIDC, backend service e middleware; o OIDC server usa Keycloak para SSO/autorização, o backend usa MySQL e o token é repassado aos demais microserviços.

## 1. Pré-requisitos de plataforma

* [ ] Validar se a versão/distribuição do Kubernetes está na matriz suportada do Portworx Central 2.11 antes de instalar.
* [ ] Garantir um provisionador block com pelo menos 307 GB livres para hospedar os PVCs usados pelos bancos do ambiente Portworx Backup/Portworx Central.
* [ ] Se o cluster também vai rodar Portworx, garantir pelo menos 50 GB livres em `/root` nos nós onde o Portworx será instalado.
* [ ] Se você pretende habilitar monitoring, reservar adicionalmente pelo menos 8 CPU cores e 16 GB de memória.
* [ ] Abrir as portas de rede exigidas para cluster internet-connected: 7070/TCP para comunicação com license server, e 8080/TCP + 8443/TCP se houver OIDC/Keycloak externo.
* [ ] Se usar provedor de autenticação externo, garantir certificados assinados por CA confiável.
* [ ] Confirmar que o Helm está instalado na máquina cliente que executará a instalação.

## 2. Decisões de desenho antes da instalação

* [ ] Se o ambiente terá PX-Central com Portworx Enterprise, instalar Portworx antes e criar uma StorageClass para o PX-Central usar.
* [ ] Se o ambiente terá PX-Central standalone, sem Portworx Enterprise, pular a etapa de instalação do Portworx.
* [ ] Definir se a autenticação ficará no Keycloak interno do PX-Central ou em um OIDC externo. No chart, o OIDC interno vem habilitado por padrão e o OIDC externo vem desabilitado por padrão.

## 3. Se for usar Portworx Enterprise no mesmo cluster

* [ ] Criar a StorageClass documentada para o PX-Central, usando o provisioner do Portworx e `repl: "3"`.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: portworx-sc
provisioner: kubernetes.io/portworx-volume
parameters:
  repl: "3"
```

## 4. Gerar a spec de instalação

* [ ] Abrir o spec generator “License Server and Monitoring” para gerar a instalação do PX-Central.
* [ ] Se estiver usando Portworx como backend, marcar `Use storage class` e informar o nome da StorageClass criada no passo anterior.
* [ ] Como o cenário é non-airgap, ignorar a configuração de custom registry; ela só é necessária para ambientes air-gapped.
* [ ] Salvar o `values.yml`/`values-px-central.yaml` gerado no passo 2 do wizard, ou usar os parâmetros `--set` gerados pelo próprio spec generator.

## 5. Parâmetros mínimos que vale revisar no chart

* [ ] Confirmar a StorageClass em `persistentStorage.storageClassName`.
* [ ] Definir senha não padrão para `pxcentralDBPassword`.
* [ ] Se usar Keycloak interno, revisar `oidc.centralOIDC.defaultUsername`, `defaultPassword` e `defaultEmail`.
* [ ] Se usar OIDC externo, preencher `oidc.externalOIDC.enabled`, `clientID`, `clientSecret` e `endpoint`.
* [ ] Ajustar `service.pxCentralUIServiceType` conforme a forma de exposição; o default do chart é `LoadBalancer`.
* [ ] Se habilitar monitoring, preencher `pxmonitor.enabled`, `pxmonitor.pxCentralEndpoint` e `pxmonitor.sslEnabled`.

## 6. Instalação

* [ ] Adicionar o repositório Helm da Portworx e atualizar o índice.

```bash
helm repo add <repo-name> portworx http://charts.portworx.io/ && helm repo update
```

* [ ] Executar a instalação do PX-Central usando o comando gerado pelo spec generator, passando o `values.yml` ou os parâmetros `--set` correspondentes.

## 7. Validação pós-instalação

* [ ] Monitorar o job de pós-instalação `pxcentral-post-install-hook` até concluir sem erro.

```bash
kubectl get po --namespace px-backup -l job-name=pxcentral-post-install-hook -o wide | awk '{print $1, $3}' | grep -iv error
```

* [ ] Se você usar o IP do master Kubernetes como endpoint do Keycloak, aplicar `iptables -P FORWARD ACCEPT` em todos os workers para permitir o encaminhamento do NodePort pelo endpoint do master.
* [ ] Confirmar que os componentes principais do PX-Central subiram normalmente no namespace de instalação.

## 8. Exposição e acesso à UI

* [ ] Validar se o modo padrão de exposição atende, já que o chart usa `LoadBalancer` por default para `px-central-ui`.
* [ ] Se preferir acesso direto por Node IP, obter `NODE_IP` e o `NODE_PORT` do serviço `px-central-ui` e acessar `http://NODE_IP:NODE_PORT`.
* [ ] Para acessar a interface do Keycloak, usar o caminho `/auth` no mesmo endpoint.

```text
http://NODE_IP:NODE_PORT
http://NODE_IP:NODE_PORT/auth
```

* [ ] Se for publicar via DNS/HTTPS com load balancer e certificados self-signed, editar o StatefulSet `pxcentral-keycloak` e definir `KC_HOSTNAME`, `KC_HOSTNAME_STRICT=true` e `KC_HOSTNAME_STRICT_HTTPS=true`.
* [ ] Se quiser operar em HTTP, editar o mesmo StatefulSet, adicionar `KC_HTTP_ENABLED=true` e remover `KC_PROXY=edge`.

## 9. Autenticação e Keycloak

* [ ] Se ficar com o OIDC interno, lembrar que o chart vem com `oidc.centralOIDC.enabled=true` por padrão.
* [ ] Se você não sobrescrever os valores, o admin padrão do Keycloak interno é `admin/admin` com email `admin@portworx.com`.
* [ ] Se habilitar OIDC externo na instalação, configurar manualmente o redirect URI no seu provedor OIDC após a instalação.
* [ ] Alterar imediatamente credenciais padrão antes de colocar o ambiente em uso administrativo.

## 10. Monitoring e license server opcionais

* [ ] Se quiser monitoring, lembrar que os parâmetros do chart incluem `pxmonitor.enabled`, `pxmonitor.pxCentralEndpoint`, `pxmonitor.sslEnabled` e tamanhos dos PVCs de Grafana/Cassandra/Consul/AlertManager/Ingester.
* [ ] Se quiser license server, abrir 7070/TCP e revisar os parâmetros `pxlicenseserver.enabled`, `pxlicenseserver.internal.enabled` ou `pxlicenseserver.external.enabled` no chart.

## 11. Adicionar o cluster ao PX-Central

* [ ] Depois de instalar o PX-Central, abrir o Lighthouse e usar a opção `Add PX Cluster` para registrar os clusters que serão gerenciados.
* [ ] No cluster que será adicionado, garantir a porta 31240 aberta para saída, porque ela é usada para enviar métricas ao Portworx Central on-prem.
* [ ] Se quiser usar os recursos Lighthouse de Prometheus/Grafana, garantir permissões para criar e configurar `ClusterRole` e `ClusterRoleBinding`; sem isso ainda dá para adicionar o cluster, mas sem os recursos de monitoramento do Lighthouse.
* [ ] Preencher o nome do cluster exatamente igual ao Portworx cluster ID obtido via `pxctl status`.
* [ ] Informar no campo `Cluster endpoint` o IP de um worker node e o port range correspondente; usar `Verify` se o port range padrão tiver sido alterado.
* [ ] Se necessário, informar `Metrics URL`, marcar `Enable metrics`, escolher o tipo de serviço Kubernetes (`EKS`, `GKE` ou `Others`) e definir o modelo de segurança do cluster (`None`, `Token` ou `OIDC`) conforme o uso de PX-Security.
* [ ] Concluir com `Submit`.

## 12. Validação final funcional

* [ ] Confirmar que o cluster aparece no Lighthouse e que a tela `Cluster Info` mostra ao menos nome do cluster, capacidade usada e quantidade de nós.
* [ ] Confirmar que a UI do PX-Central está apta a monitorar e gerenciar clusters Portworx, visualizar volumes e operar como ponto central de gestão.

## 13. Checklist de encerramento

* [ ] StorageClass validada.
* [ ] values gerado e salvo.
* [ ] Helm repo atualizado.
* [ ] Instalação executada.
* [ ] Post-install hook validado.
* [ ] UI acessível.
* [ ] Keycloak ajustado conforme HTTP/HTTPS.
* [ ] Credenciais administrativas trocadas.
* [ ] OIDC externo configurado, se aplicável.
* [ ] Monitoring habilitado, se aplicável.
* [ ] License server habilitado, se aplicável.
* [ ] Cluster adicionado no Lighthouse.
* [ ] Dashboards e visibilidade do ambiente confirmados.


---

## Sources

- [Install Portworx Central on-premises | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/install/px-central)
- [Install chart reference | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/install/install-chart-reference)
- [Operate Portworx using Portworx Central on-premises | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/)
- [Install Portworx Central on-premises in air-gapped environment | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/install/air-gapped-install)
- [Configure access to UI | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/px-central-operations/configure-ui)
- [Add clusters to Portworx Central on-premises | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/px-central-operations/add-clusters)
