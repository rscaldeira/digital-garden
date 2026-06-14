---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/px-backup/checklist_instalacao_px_backup/","dg-note-properties":{}}
---

# Checklist de instalação e configuração do PX-Backup

## Escopo e premissas

* [ ] Este checklist cobre o PX-Backup on-prem 3.0.0 e assume como caminho principal a instalação via Portworx Central Spec Generator. A landing page pública do produto já está na versão 3.0.0, mas a matriz pública de pré-requisitos que consegui confirmar ainda lista OpenShift 4.20.x, 4.19.x e 4.18.x; por isso, o documento não assume suporte oficial a 4.21.x sem validação adicional no support matrix vigente e no Spec Generator. O PX-Backup pode ser instalado em qualquer cluster Kubernetes suportado, seja um cluster dedicado ou um dos próprios application clusters. Não é necessário instalar o PX-Backup em todos os clusters que serão protegidos; depois da instalação, os demais clusters podem ser adicionados pela web console.
* [ ] Se o ambiente for non-airgap, escolha entre os métodos suportados: Portworx Central Spec Generator ou Argo CD. Este checklist detalha o fluxo com Spec Generator.
* [ ] Se o ambiente for airgap, será necessário fluxo separado com registry privado e preparação de imagens; este checklist não detalha esse cenário.

## 1. Pré-requisitos de plataforma

* [ ] Garantir pelo menos 3 worker nodes no cluster onde o PX-Backup será instalado.
* [ ] Validar se a distribuição/versão do Kubernetes está na matriz suportada. A documentação cita, entre outros, Vanilla Kubernetes 1.34/1.33/1.32, OpenShift 4.20/4.19/4.18, RKE2, TKGi, TKGs, Charmed Kubernetes, AKS, EKS e GKE em versões suportadas.
* [ ] Dimensionar CPU, RAM e storage conforme a quantidade de backups esperada:
  * até 2.000 backups: 4 vCPU mínimas por nó, 4 GB RAM por nó e 506 GB totais para PVCs.
  * de 2.000 a 10.000 backups: 8 vCPU por nó, 8 GB RAM por nó e 506 GB totais.
  * de 10.000 a 15.000 backups: 8 vCPU por nó, 16 GB RAM por nó e 506 GB totais.
  * até 75.000 backups: 32 vCPU por nó, 32 GB RAM por nó e 506 GB totais.
* [ ] Garantir um provisionador block com pelo menos 307 GB livres para os PVCs usados pelos bancos do PX-Backup.
* [ ] Se houver Portworx no cluster, garantir pelo menos 50 GB livres em `/root` nos nós onde o Portworx será instalado.
* [ ] Confirmar que a máquina cliente usada para a instalação tem Helm instalado.
* [ ] Se for usar provider de autenticação externo, garantir certificados assinados por uma CA confiável.
* [ ] Se houver necessidade de KDMP ou cross-cloud backups, reservar buffer storage adicional equivalente ao dobro do tamanho do PVC original da aplicação.
* [ ] Se já existir Prometheus cluster-wide, garantir que ele não reconcilie os recursos Prometheus/Alertmanager do namespace do PX-Backup.
* [ ] Validar que os 3 pods do replica set do MongoDB poderão ser agendados e resolvidos por DNS; a partir do PXB 2.9 isso é obrigatório para o serviço subir corretamente.
* [ ] Validar suporte a AVX no hardware se estiver implantando PXB 2.3 ou superior, por causa do MongoDB 5.x interno.

## 2. Decisões antes da instalação

* [ ] Definir o namespace do PX-Backup e a StorageClass antes de iniciar. A documentação informa que namespace e StorageClass são configuração de uso único na instalação inicial e não podem ser alterados depois no upgrade.
* [ ] Definir se a autenticação ficará apenas no fluxo padrão do produto ou se haverá integração com OIDC externo.
* [ ] Definir se será usado Prometheus existente ou o stack padrão do PX-Backup.
* [ ] Definir se há proxy no ambiente e, se houver, se é sem autenticação ou com autenticação/CA.
* [ ] Definir se o cluster usa service mesh. Se houver Istio sidecar ou Linkerd, isso precisa ser refletido no comando Helm/values file.

## 3. Preparar os parâmetros de instalação

* [ ] Planejar os segredos do ambiente: MySQL root password, Postgres user password, MongoDB PX-Backup user password, MongoDB root password, MongoDB replica set key e, se habilitar encryption at rest do MongoDB, a master encryption key.
* [ ] Guardar com segurança a chave de criptografia do MongoDB. Se ela for perdida, os dados criptografados ficam inacessíveis, e a documentação informa que essas chaves não podem ser rotacionadas depois de definidas.
* [ ] Garantir que as senhas definidas seguem a política de password do produto.

## 4. Gerar a instalação no Portworx Central SpecGen

* [ ] Acessar o Portworx Central SpecGen do PX-Backup.
* [ ] Fazer login, aceitar o EULA e, se necessário, criar conta.
* [ ] Na aba Spec Details, preencher:
  * versão do PX-Backup;
  * namespace;
  * tipo de ambiente (On-Premises ou Cloud);
  * StorageClass name.
* [ ] Se necessário, habilitar Rancher RBAC somente quando o ambiente usar Rancher e provider compatível.
* [ ] Se necessário, preencher integração com OIDC externo: endpoint, client ID e client secret.
* [ ] Se necessário, preencher integração com Prometheus/Alertmanager existentes.
* [ ] Se necessário, preencher proxy configuration.
* [ ] Em ambiente non-airgap, não usar custom registry; isso é apenas para airgap.

## 5. Executar a instalação

* [ ] Na aba Finish, executar primeiro o Step 1 para criar o secret `pxc-credentials` com credenciais e parâmetros de configuração.
* [ ] Ainda na aba Finish, executar o comando do Step 2 para adicionar o Helm repo e atualizá-lo.
* [ ] Escolher o modo de instalação:
  * opção default com comando `--set`;
  * opção avançada usando `values-px-central.yaml`.
* [ ] Se usar Istio sidecar, habilitar `istio.enabled=true`; se usar Linkerd, habilitar `linkerd.enabled=true`.
* [ ] Se houver múltiplas aplicações compartilhando o prefixo `/` com Istio sidecar, definir `hostName`/`istio.hostName`; nesse caso ele é obrigatório para evitar conflito de rota.
* [ ] Não usar `--no-hooks` no `helm install`, porque a documentação alerta que isso pode deixar o cluster em bad state.
* [ ] Se usar o values file, baixar o `values-px-central.yaml`, ajustar os parâmetros necessários, salvar e instalar com ele.
* [ ] Executar o install final do PX-Backup.

## 6. Validar a instalação

* [ ] Acompanhar o post-install hook `pxcentral-post-install-hook` até concluir com sucesso.

```bash
kubectl get pod --namespace <pxb-namespace> -ljob-name=pxcentral-post-install-hook -o wide | awk '{print $1, $3}' | grep -iv error
kubectl get job pxcentral-post-install-hook -n <pxb-namespace> -w
```

* [ ] Validar que todos os pods do PX-Backup estão em `Running` ou `Completed`.

```bash
kubectl get pods -n <pxb-namespace>
```

* [ ] Se a instalação falhar, revisar os health checks e mensagens do CLI, porque o produto executa uma verificação de requisitos após o install.

## 7. Configuração pós-instalação

* [ ] Abrir as portas 10001, 10002, 10005 e 10006 no backup cluster.
* [ ] Se desejar, integrar um provider externo de autorização após a instalação. A documentação cita Azure AD, OpenLDAP, External Keycloak e OpenShift v4.
* [ ] Preparar os application clusters:
  * com Portworx Enterprise, validar instalação do Portworx e upgrade do Stork para uma versão compatível com o PX-Backup;
  * sem Portworx Enterprise, instalar ou atualizar o Stork separadamente.
* [ ] Recuperar as credenciais administrativas iniciais a partir do secret do Keycloak.

```bash
kubectl get secret pxcentral-keycloak-http -n <pxb-namespace> -o jsonpath="{.data.username}" | base64 --decode && echo
kubectl get secret pxcentral-keycloak-http -n <pxb-namespace> -o jsonpath="{.data.password}" | base64 --decode && echo
```

* [ ] Fazer o primeiro login com `admin` e a senha gerada na instalação.
* [ ] Trocar imediatamente a senha no primeiro acesso e armazená-la de forma segura. A recomendação é usar ao menos 12 caracteres com maiúsculas, minúsculas, números e caracteres especiais.
* [ ] Aplicar a licença do Portworx Backup se quiser sair do trial e ativar enterprise features.

## 8. Checklist de aceite final

* [ ] PX-Backup instalado em namespace definitivo.
* [ ] StorageClass definitiva usada na instalação.
* [ ] Secret `pxc-credentials` criado com sucesso.
* [ ] Helm repo adicionado e install executado.
* [ ] `pxcentral-post-install-hook` concluído sem erro.
* [ ] Pods do PX-Backup em `Running`/`Completed`.
* [ ] Portas pós-instalação liberadas no backup cluster.
* [ ] Web console acessível.
* [ ] Credenciais admin recuperadas e senha inicial trocada.
* [ ] Stork instalado/atualizado conforme o tipo de application cluster.
* [ ] Licença aplicada, se aplicável.
* [ ] Application clusters prontos para serem adicionados e protegidos pela console do PX-Backup.


---

## Sources

- [Install Portworx Backup on an On-premise or Cloud Cluster | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install)
- [Install Prerequisites | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install-prereq)
- [Install Portworx Backup Using Portworx Central | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install/install-procedure)
- [Set up Portworx Backup for Operations | Portworx Backup On-Premises Documentation](https://docs.portworx.com/portworx-backup-on-prem/install/install/set-up-pxb-operations)