---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/px-central/runbook_px_central_openshift_metallb/","dg-note-properties":{}}
---

# Runbook de instalação do PX-Central on-prem em OpenShift com MetalLB

## 1. Escopo deste runbook

Este runbook foi montado para o seguinte cenário: OpenShift suportado, Portworx já instalado no cluster, ambiente non-airgap, publicação da UI via service `LoadBalancer` atendido pelo MetalLB, Keycloak interno do PX-Central, com License Server e Monitoring habilitados.

Para Portworx Central 2.11, o OpenShift suportado é 4.20.x, 4.19.x e 4.18.x.

O cluster mínimo suportado é de 3 worker nodes.

Se você habilitar Monitoring, a documentação pede adicionalmente pelo menos 8 CPU cores e 16 GB de memória além dos requisitos base.

## 2. Variáveis que você deve definir antes de começar

Preencha estes valores antes da execução:

```bash
export PX_NS="px-backup"
export PX_SC="px-central-repl3"
export PX_HELM_REPO_NAME="portworx"
export PX_RELEASE_NAME="px-central"
export PX_UI_HOST="px-central.seudominio.local"
export PX_UI_URL="https://px-central.seudominio.local"
export PX_ADMIN_EMAIL="storage-admin@seudominio.local"
export PX_DB_PASSWORD="TROCAR_SENHA_DB"
export PX_KEYCLOAK_ADMIN="pxadmin"
export PX_KEYCLOAK_PASSWORD="TROCAR_SENHA_KEYCLOAK"
export PX_LICENSE_ADMIN="admin"
export PX_LICENSE_PASSWORD="TROCAR_SENHA_LICENSE"
```

## 3. Pré-check de plataforma

Valide se o ambiente atende aos pré-requisitos abaixo antes da instalação.

- OpenShift em versão suportada.
- Pelo menos 307 GB livres em provisionador block para PVCs do ambiente.
- Como o cluster já roda Portworx, garantir pelo menos 50 GB livres em `/root` nos nós onde o Portworx está instalado.
- Portas abertas para cluster internet-connected:
    - 7070/TCP para License Server.
    - 8080/TCP e 8443/TCP somente se você for usar OIDC externo.
- Helm instalado na máquina cliente.

Exemplo de validação operacional:

```bash
oc version
helm version
oc get nodes -o wide
oc get sc
```

## 4. StorageClass para o PX-Central

Como o seu cluster já tem Portworx, a documentação orienta criar uma StorageClass e depois referenciá-la na instalação do PX-Central.

Se você ainda não tiver uma SC dedicada, use o exemplo oficial abaixo:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: px-central-repl3
provisioner: kubernetes.io/portworx-volume
parameters:
  repl: "3"
```

Aplicação:

```bash
oc apply -f px-central-repl3.yaml
oc get sc ${PX_SC}
```

## 5. Gerar a spec oficial da instalação

A instalação deve ser gerada no spec generator “License Server and Monitoring”.

No wizard:

- Selecione o cenário com License Server e Monitoring.
- Marque `Use storage class`.
- Informe `px-central-repl3` ou o nome da SC que você criou.
- Como o seu ambiente é non-airgap, não use custom registry.

Ao final, salve o `values.yml` exibido na etapa `Complete`. A própria documentação manda usar o comando ou o arquivo gerado nessa etapa para instalar o PX-Central.

## 6. Exemplo de values.yaml para o seu cenário

Use o arquivo do wizard como fonte principal. O exemplo abaixo serve como base para revisão e alinhamento dos parâmetros mais importantes do seu cenário.

```yaml
persistentStorage:
  enabled: true
  storageClassName: px-central-repl3
  mysqlVolumeSize: 100Gi
  keycloakThemeVolumeSize: 5Gi
  keycloakBackendVolumeSize: 10Gi

pxcentralDBPassword: "TROCAR_SENHA_DB"

oidc:
  centralOIDC:
    enabled: true
    defaultUsername: "pxadmin"
    defaultPassword: "TROCAR_SENHA_KEYCLOAK"
    defaultEmail: "storage-admin@seudominio.local"
  externalOIDC:
    enabled: false

service:
  pxCentralUIServiceType: LoadBalancer

pxlicenseserver:
  enabled: true
  internal:
    enabled: true
  external:
    enabled: false
  adminUserName: "admin"
  adminUserPassword: "TROCAR_SENHA_LICENSE"

pxmonitor:
  enabled: true
  pxCentralEndpoint: "https://px-central.seudominio.local"
  sslEnabled: true
```

Observações importantes sobre esse arquivo:

- O OIDC interno do PX-Central vem habilitado por padrão.
- O OIDC externo vem desabilitado por padrão.
- O chart default da UI usa `LoadBalancer`, o que combina com o seu cenário de MetalLB.
- O chart default traz credenciais padrão em alguns itens; troque isso antes de colocar em uso administrativo.

## 7. Adicionar o repositório Helm

```bash
helm repo add ${PX_HELM_REPO_NAME} portworx http://charts.portworx.io/ && helm repo update
```

## 8. Instalar o PX-Central

A documentação manda instalar usando exatamente o comando ou o `values.yml` gerado pelo spec generator.

Fluxo recomendado:

1. Salve o conteúdo do wizard em `values-px-central.yaml`.
2. Execute o comando Helm gerado na aba `Complete`.
3. Se preferir, mantenha o comando do wizard e apenas substitua os segredos pelos valores finais do ambiente.

Exemplo operacional de estrutura, apenas como referência de execução:

```bash
helm install ${PX_RELEASE_NAME} <chart-name-from-wizard> \
  -n ${PX_NS} \
  --create-namespace \
  -f values-px-central.yaml
```

Use `${PX_RELEASE_NAME}` como nome local do release. O valor `<chart-name-from-wizard>` deve ser copiado do comando exibido na aba `Complete` do spec generator.

Se o comando do wizard vier em formato `--set`, prefira usar o comando do wizard sem simplificar, porque ele é o formato oficialmente entregue pela própria documentação para aquele cenário.

## 9. Validar o pós-install

A validação oficial inicial é acompanhar o job `pxcentral-post-install-hook`.

```bash
kubectl get po --namespace px-backup -l job-name=pxcentral-post-install-hook -o wide | awk '{print $1, $3}' | grep -iv error
```

Validações adicionais úteis:

```bash
oc -n ${PX_NS} get pods
oc -n ${PX_NS} get svc
oc -n ${PX_NS} get pvc
```

## 10. Publicar a UI usando MetalLB

Como o chart usa `LoadBalancer` por padrão para a UI, o caminho natural no seu ambiente é deixar o service `px-central-ui` ser atendido pelo MetalLB.

Valide o IP externo:

```bash
oc -n ${PX_NS} get svc px-central-ui
```

Quando o `EXTERNAL-IP` estiver atribuído, publique o DNS apontando para esse IP.

Se você quiser acessar por NodePort temporariamente, a documentação também suporta acesso por `NODE_IP:NODE_PORT` e a UI do Keycloak em `/auth`.

```text
http://NODE_IP:NODE_PORT
http://NODE_IP:NODE_PORT/auth
```

## 11. Ajuste do Keycloak interno

Como você está usando Keycloak interno, não precisa configurar OIDC externo neste momento.

### Cenário recomendado: acesso por DNS/HTTPS

Se a UI for publicada por DNS/HTTPS, edite o StatefulSet do Keycloak e ajuste o hostname.

```bash
oc -n ${PX_NS} edit sts pxcentral-keycloak
```

Adicione ou ajuste as variáveis:

```yaml
- name: KC_HOSTNAME
  value: px-central.seudominio.local
- name: KC_HOSTNAME_STRICT
  value: "true"
- name: KC_HOSTNAME_STRICT_HTTPS
  value: "true"
```

### Cenário alternativo: acesso em HTTP

Se você quiser operar a UI em HTTP, a documentação manda habilitar `KC_HTTP_ENABLED=true` e remover `KC_PROXY=edge`.

```yaml
- name: KC_HTTP_ENABLED
  value: "true"
```

E remova:

```yaml
- name: KC_PROXY
  value: edge
```

## 12. Troca de credenciais padrão

O chart default traz valores padrão para o admin do Keycloak interno e para o banco do PX-Central; eles devem ser substituídos antes de uso administrativo.

Defaults relevantes do chart:

- `pxcentralDBPassword`: `Password1`.
- `oidc.centralOIDC.defaultUsername`: `admin`.
- `oidc.centralOIDC.defaultPassword`: `admin`.
- `oidc.centralOIDC.defaultEmail`: `admin@portworx.com`.
- `pxlicenseserver.adminUserName`: `admin`.
- `pxlicenseserver.adminUserPassword`: `Adm1n!Ur`.

## 13. Validação funcional do License Server e Monitoring

Como você habilitou License Server e Monitoring, confira se os parâmetros abaixo foram realmente aplicados no arquivo final e se os pods correspondentes subiram.

```bash
oc -n ${PX_NS} get pods | egrep 'license|grafana|cassandra|consul|alert|ingester|prometheus|pxcentral'
```

Pontos de revisão:

- `pxlicenseserver.enabled: true`.
- `pxlicenseserver.internal.enabled: true`.
- `pxmonitor.enabled: true`.
- `pxmonitor.pxCentralEndpoint` apontando para o endpoint real da UI.
- `pxmonitor.sslEnabled` coerente com o modo HTTP ou HTTPS que você escolheu.

## 14. Caso use IP do master como endpoint do Keycloak

Se, por algum motivo, você usar o IP do master Kubernetes como endpoint do Keycloak, a documentação pede o ajuste abaixo em todos os workers para habilitar o encaminhamento do NodePort via endpoint do master.

```bash
sudo iptables -P FORWARD ACCEPT
```

## 15. Adicionar o cluster no Lighthouse

Depois de instalar o PX-Central, o próximo passo funcional é registrar o cluster no Lighthouse.

Pré-requisitos do cluster a ser adicionado:

- Porta 31240 liberada para saída, usada para enviar métricas ao Portworx Central on-prem.
- Permissão para criar `ClusterRole` e `ClusterRoleBinding` se você quiser usar os recursos de monitoramento Lighthouse com Prometheus/Grafana.

No cadastro do cluster:

- O nome do cluster deve ser exatamente o Portworx cluster ID.
- Para descobrir o ID, rode `pxctl status` em um nó com Portworx.
- Em `Cluster endpoint`, informe o IP de um worker node e o port range correspondente.
- Se for usar segurança com PX-Security, escolha `None`, `Token` ou `OIDC` conforme o seu modelo.
- Finalize em `Submit`.

## 16. Critérios de aceite finais

Considere a instalação concluída quando estes itens estiverem OK:

- Pods do PX-Central em estado estável no namespace de instalação.
- `pxcentral-post-install-hook` concluído sem erro.
- Service `px-central-ui` com IP externo entregue pelo MetalLB.
- UI acessível pelo hostname final.
- `/auth` acessível para a interface do Keycloak.
- Credenciais padrão substituídas.
- License Server e Monitoring ativos.
- Cluster adicionado ao Lighthouse e visível em `Cluster Info` com nome, capacidade usada e número de nós.

---

## Sources

- [Install Portworx Central on-premises | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/install/px-central)
- [Install chart reference | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/install/install-chart-reference)
- [Configure access to UI | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/px-central-operations/configure-ui)
- [Add clusters to Portworx Central on-premises | Portworx Central On-Premises Documentation](https://docs.portworx.com/portworx-central-on-prem/px-central-operations/add-clusters)

&nbsp;