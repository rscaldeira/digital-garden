---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/runbook_px_backup_openshift_metallb/","dg-note-properties":{}}
---

# Runbook de instalação do PX-Backup on-prem em OpenShift

## 1. Escopo deste runbook

Este runbook foi montado para o cenário de PX-Backup 3.0.0 em OpenShift com Portworx já instalado, ambiente non-airgap, uso de Prometheus e Alertmanager já existentes e publicação da UI via LoadBalancer atendido pelo MetalLB. A landing page pública do produto já está na versão 3.0.0, mas a matriz pública de pré-requisitos que consegui confirmar ainda lista OpenShift 4.20.x, 4.19.x e 4.18.x; por isso, este runbook não assume suporte oficial a 4.21.x sem validação adicional no support matrix vigente e no Spec Generator.

O PX-Backup pode ser instalado em um cluster dedicado ou em um dos próprios application clusters, e você não precisa instalá-lo em todos os clusters que serão protegidos; os demais podem ser adicionados depois pela web console.

Para ambiente non-airgap, a documentação suporta instalação via Portworx Central Spec Generator ou Argo CD. Neste runbook, o caminho usado é o Spec Generator.

## 2. Variáveis que vale definir antes de começar

Preencha estes valores antes da execução:

```bash
export PXB_NS="px-backup"
export PXB_SC="px-backup-repl3"
export PXB_HELM_REPO_NAME="portworx"
export PXB_RELEASE_NAME="px-central"
export PXB_VERSION="3.0.0"

export PXB_PROM_ENDPOINT="https://prometheus.seudominio.local"
export PXB_ALERT_ENDPOINT="https://alertmanager.seudominio.local"
export PXB_PROM_SECRET="prometheus-secret"
export PXB_ALERT_SECRET="alertmanager-secret"

export PXB_UI_HOST="px-backup.seudominio.local"
```

Você pode definir desde já o namespace, o nome da StorageClass, o alias do Helm repo e o release name. O ponto que deve vir do Spec Generator é o comando final da aba Finish, porque ele incorpora os parâmetros e segredos gerados para o seu cenário.

O exemplo oficial do fluxo com values file continua usando o chart `portworx/px-central` e um release exemplo `px-central`, mesmo quando o objetivo é instalar PX-Backup.

## 3. Pré-check de plataforma

Valide estes itens antes de começar:

- O cluster onde o PX-Backup será instalado deve ter no mínimo 3 worker nodes.
- O sizing mínimo depende da quantidade esperada de backups. Até 2.000 backups, a base é 4 vCPU e 4 GB RAM por nó, com 506 GB totais para PVCs. Entre 2.000 e 10.000 backups, use 8 vCPU e 8 GB RAM por nó. Entre 10.000 e 15.000 backups, use 8 vCPU e 16 GB RAM por nó. Até 75.000 backups, use 32 vCPU e 32 GB RAM por nó, sempre mantendo 506 GB totais para PVCs.
- Garanta pelo menos 307 GB livres em um provisionador block para os PVCs usados pelos bancos do PX-Backup.
- Como o cluster já roda Portworx, garanta também pelo menos 50 GB livres em `/root` nos nós onde o Portworx está instalado.
- Helm deve estar instalado na máquina cliente que vai rodar a instalação.
- Se você usar auth provider externo, os certificados devem ser assinados por uma CA confiável.
- Se pretende usar KDMP ou cross-cloud backup, reserve buffer storage adicional equivalente ao dobro do tamanho do PVC original da aplicação.
- Desde o PXB 2.9, os 3 pods do replica set do MongoDB precisam ficar DNS-resolvable e acessíveis na subida; caso contrário, o PX-Backup pode não subir corretamente.
- A partir do PXB 2.3, o MongoDB interno exige suporte a AVX no hardware Intel/AMD.

Validação operacional inicial:

- A partir do PXB 2.3, o MongoDB interno exige suporte a AVX no hardware Intel/AMD.

Validação sugerida do requisito de AVX:

```bash
lscpu | grep -i avx
cat /proc/cpuinfo | grep avx
```

```bash
oc version
helm version
oc get nodes -o wide
oc get sc
```

## 4. Preparar a StorageClass do PX-Backup

Como o seu cluster já tem Portworx, a documentação manda instalar Portworx primeiro e depois criar uma StorageClass para o PX-Backup usar. O exemplo oficial usa `portworx-sc`; neste runbook, o nome foi ajustado para `px-backup-repl3`, o que é válido desde que você use exatamente o mesmo nome no Spec Generator e na instalação final.

Use esta StorageClass:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: px-backup-repl3
provisioner: pxd.portworx.com
parameters:
  repl: "3"
```

Aplicação:

```bash
oc apply -f px-backup-repl3.yaml
oc get sc ${PXB_SC}
```

## 5. Preparar o ambiente com Prometheus e Alertmanager existentes

Como você escolheu usar Prometheus e Alertmanager já existentes, valide primeiro se as versões estão dentro do que a matriz pública de pré-requisitos atualmente lista para a documentação consultada: Prometheus 2.35.0, Prometheus Operator 0.87.1 e Alertmanager 0.30.0, 0.28.0, 0.27.0 ou 0.26.0.

No Spec Generator, você deverá marcar `Use existing Prometheus` e informar:

- Prometheus Endpoint
- Alertmanager Endpoint
- Prometheus secret name
- Alertmanager secret name

Se houver Prometheus cluster-wide, a documentação também pede cuidado para que ele não reconcilie os recursos do PX-Backup no namespace `px-backup`. Se houver conflito ou CrashLoopBackOff em Prometheus/Alertmanager, ajuste o Prometheus Operator cluster-wide para excluir o namespace do PX-Backup com `--deny-namespaces=<pxb-namespace>` ou então restrinja o escopo com `--namespaces=<cluster-wide-prometheus-installed-namespace>`.

## 6. Pré-instalação obrigatória

Antes de abrir o wizard, passe por estes itens:

- Revise a password policy do produto.
- Defina o namespace definitivo e a StorageClass definitiva já na instalação inicial. A documentação diz que namespace e StorageClass são configuração de uma vez só, porque os PVCs serão criados com esses valores e eles não podem ser alterados depois no upgrade.
- Defina previamente as senhas/segredos de MySQL, PostgreSQL, MongoDB PX-Backup user, MongoDB root, MongoDB replica set key e, se você habilitar encriptação at rest do MongoDB, também a master encryption key.
- Se for usar encrypt at rest no MongoDB, trate a master encryption key como dado crítico. A documentação alerta que a perda dessa chave torna os dados inacessíveis e que a chave não pode ser rotacionada depois de definida.

## 7. Gerar a instalação no Portworx Central Spec Generator

Use o fluxo do Spec Generator e preencha os campos assim:

1. Acesse o fluxo de instalação do PX-Backup usando Portworx Central.
2. Em `Backup Version`, selecione a versão desejada.
3. Em `Namespace`, informe `${PXB_NS}`.
4. Em `Select your environment`, escolha `On-Premises`.
5. Em `StorageClass Name`, informe `${PXB_SC}`.
6. Em `Use existing Prometheus`, informe `${PXB_PROM_ENDPOINT}`, `${PXB_ALERT_ENDPOINT}`, `${PXB_PROM_SECRET}` e `${PXB_ALERT_SECRET}`.
7. Não marque `Use custom registry`, porque isso é aplicável apenas para ambientes air-gapped.
8. Se não houver Auth0 externo, deixe `Use your OIDC` desmarcado. A documentação cita esse bloco apenas para integração com Auth0 externo.
9. Preencha as credenciais de banco na seção `Database Credentials`.

Para este cenário, prefira a opção com `values-px-central.yaml` em vez do `--set`, porque você está combinando OpenShift, Portworx pré-existente, Prometheus/Alertmanager externos e publicação externa da UI.

## 8. Instalar o PX-Backup

Na aba `Finish`, siga esta ordem:

### 8.1. Executar o Step 1

Copie e execute o comando do Step 1 exatamente como o Spec Generator gerar. Ele cria o namespace e o secret `pxc-credentials` com as credenciais e parâmetros de banco em base64.

A documentação mostra que esse secret inclui:

- `mongodb-px-backup-password`
- `mongodb-root-password`
- `mongodb-replica-set-key`
- `mongodb-master-encryption-key`
- `postgresql-password`
- `mysql-password`

### 8.2. Adicionar o Helm repo

Execute:

```bash
helm repo add ${PXB_HELM_REPO_NAME} http://charts.portworx.io/ && helm repo update
```

Esse é o comando oficial do fluxo non-airgap.

### 8.3. Baixar e usar o values file

Baixe o `values-px-central.yaml` fornecido pelo próprio wizard e, se quiser padronizar no seu diretório de trabalho, renomeie para:

```bash
values-px-central-${PXB_VERSION}.yaml
```

A própria documentação descreve esse fluxo como a opção avançada recomendada quando você quer controlar melhor os parâmetros.

### 8.4. Executar o helm install final

Use o comando da aba `Finish` como fonte de verdade. A estrutura esperada será semelhante a esta:

```bash
helm install ${PXB_RELEASE_NAME} portworx/px-central \
  --namespace ${PXB_NS} \
  --create-namespace \
  --version ${PXB_VERSION} \
  -f values-px-central-${PXB_VERSION}.yaml
```

O ponto importante aqui é não “inventar” parâmetros fora do Spec Generator. A documentação é explícita de que os parâmetros preenchidos em `Spec Details` são incorporados ao comando ou ao values file gerado no `Finish`.

## 9. Observações específicas de service mesh

Se o cluster do PX-Backup usar Istio sidecar, a documentação manda adicionar `istio.enabled=true` e, se houver múltiplas aplicações usando o prefixo `/`, o parâmetro `istio.hostName` se torna obrigatório para evitar conflito de rota.

Se usar Linkerd, a documentação pede `linkerd.enabled=true` e as anotações de namespace correspondentes.

Como o seu cenário informado não depende disso, o runbook assume service mesh desabilitado.

## 10. Validação do deploy

Depois do `helm install`, acompanhe o post-install hook:

```bash
kubectl get pod --namespace ${PXB_NS} -ljob-name=pxcentral-post-install-hook -o wide | awk '{print $1, $3}' | grep -iv error
kubectl get job pxcentral-post-install-hook -n ${PXB_NS} -w
```

A documentação também informa que, se a instalação falhar por health check, as mensagens relevantes aparecem no CLI após o install.

Validações operacionais úteis:

```bash
oc -n ${PXB_NS} get pods
oc -n ${PXB_NS} get svc
oc -n ${PXB_NS} get pvc
```

Considere a instalação tecnicamente estável quando os pods principais estiverem em `Running` ou `Completed` e o post-install hook tiver terminado sem erro.

## 11. Publicação da UI com LoadBalancer / MetalLB

Este runbook assume que a UI do PX-Backup será publicada no seu padrão de cluster usando Service `LoadBalancer` com IP anunciado pelo MetalLB.

Depois do deploy:

```bash
oc -n ${PXB_NS} get svc
```

Quando o serviço da UI receber `EXTERNAL-IP`, publique o DNS desejado, por exemplo `${PXB_UI_HOST}`, apontando para esse IP. Depois disso, acesse a web console pelo hostname final.

## 12. Primeiro acesso e troca de senha inicial

Depois de configurar o acesso à web console, recupere a senha aleatória do usuário admin com o comando abaixo:

```bash
kubectl get secret pxcentral-keycloak-http -n ${PXB_NS} -o jsonpath="{.data.password}" | base64 --decode && echo
```

Use essa senha no primeiro login na web console. A documentação informa que, no primeiro acesso, você será obrigado a trocar a senha.

A recomendação oficial é usar uma senha forte com pelo menos 12 caracteres, incluindo maiúsculas, minúsculas, números e caracteres especiais, e armazená-la de forma segura.

## 13. Licenciamento após a instalação

Ao concluir a instalação pelo wizard, o PX-Backup fica em modo trial. Para passar ao enterprise, aplique a licença depois do primeiro acesso.

No ambiente non-airgap, a documentação orienta o fluxo:

1. Login na web console.
2. `Settings` no menu lateral.
3. `License`.
4. `Details`.
5. `Import License`.
6. Escolher `License Key` para fixed node count ou `Subscription` para node usage-based licensing.

## 14. Critérios de aceite finais

Considere este runbook concluído quando estes itens estiverem OK:

- Namespace do PX-Backup criado e secret `pxc-credentials` aplicado.
- StorageClass `px-backup-repl3` criada e usada na instalação.
- Helm repo adicionado e cache atualizado.
- Comando final gerado pelo Spec Generator executado com sucesso.
- Post-install hook `pxcentral-post-install-hook` concluído sem erro.
- Pods principais do namespace em estado estável.
- Web console acessível pelo endpoint publicado no LoadBalancer.
- Senha inicial do admin recuperada e trocada no primeiro login.
- Prometheus/Alertmanager externos preenchidos corretamente no Spec Generator e sem conflito de reconciliação com o namespace do PX-Backup.
- Licença aplicada, se o ambiente já for sair do trial.

---

## Sources

- [Install Prerequisites | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install-prereq)
- [Install Portworx Backup on an On-premise or Cloud Cluster | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install)
- [Install PXB on Non-Airgapped Environments](https://docs.portworx.com/portworx-backup-on-prem/install/install/non-airgapped/non-airgapped-install)
- [Pre-installation | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install/non-airgapped/pre-install)
- [Install Portworx Backup on Airgapped Environments](https://backup.docs.portworx.com/install/air-gapped-install/)
- [Post-installation | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install/non-airgapped/post-install)
- [Add a Portworx Backup License in a Non-Air-Gapped Environment](https://docs.portworx.com/portworx-backup-on-prem/install/backup-licenses/deploy-license)

&nbsp;