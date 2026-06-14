---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray Configuration/","dg-note-properties":{}}
---


# **FlashArray Configuration**

## **Objetivo**

Documentar as informações e configurações necessárias no FlashArray para integração com Portworx Enterprise no Lab POA.

Este documento deve ser utilizado durante a instalação inicial e também em futuras reconstruções do ambiente.

---

## **Informações Necessárias**

### **FlashArray**

- Modelo:
- Purity Version:
- Management IP:
- Array Name:

---

## **Conectividade**

Definir o protocolo utilizado entre os nodes OpenShift e o FlashArray.

Opções esperadas para o Lab POA:

- Fibre Channel
- iSCSI

Protocolo definido:

- Fibre Channel
- iSCSI

---

# **Fibre Channel**

## **Pré-Requisitos**

- HBAs instalados nos servidores
- WWPNs identificados
- Zoning configurado entre servidores e FlashArray
- Portas FC do FlashArray ativas
- Multipath configurado nos nodes OpenShift

---

## **Informações a Registrar**

|**Node**|**HBA Port 1 WWPN**|**HBA Port 2 WWPN**|
|---|---|---|
|node01|||
|node02|||
|node03|||

---

## **Validação nos Nodes**

Executar nos nodes:

```bash
systool -c fc_host -v
```

Validar:

- HBAs visíveis
- WWPNs corretos
- Link state online

---

## **Validação de Multipath**

Executar:

```bash
multipath -ll
```

Resultado esperado:

- Caminhos do FlashArray visíveis
- Múltiplos paths por volume
- Sem paths failed

---

# **iSCSI**

## **Pré-Requisitos**

- Interfaces de storage configuradas
- VLAN iSCSI definida
- Portas iSCSI do FlashArray acessíveis pelos nodes
- Serviço iscsid habilitado nos nodes
- Initiator name único por node

---

## **Informações a Registrar**

|**Node**|**iSCSI Initiator Name**|**IP Storage**|
|---|---|---|
|node01|||
|node02|||
|node03|||

---

## **Portas iSCSI FlashArray**

|**FlashArray Port**|**IP**|
|---|---|
|ct0.ethX||
|ct1.ethX||

---

## **Validação de Conectividade**

Executar a partir dos nodes:

```bash
ping <flasharray_iscsi_ip>
```

Opcionalmente validar discovery:

```bash
iscsiadm -m discovery -t sendtargets -p <flasharray_iscsi_ip>
```

---

# **Usuário e API Token**

## **Objetivo**

Criar uma conta no FlashArray para uso do Portworx.

Requisito:

- Usuário com role Storage Admin
- API Token gerado para este usuário

---

## **Informações a Registrar**

|**Campo**|**Valor**|
|---|---|
|Usuário||
|Role|Storage Admin|
|API Token|Não registrar token em texto claro|
|Data de criação||
|Responsável||

---

# **pure.json**

O Portworx utiliza o arquivo `pure.json` para se comunicar com o FlashArray.

Exemplo:

```json
{
  "FlashArrays": [
    {
      "MgmtEndPoint": "x.x.x.x",
      "APIToken": "TOKEN"
    }
  ]
}
```

Observação:

Não armazenar o API Token real no Obsidian.

Usar placeholder nos documentos.

---

# **Kubernetes Secret**

O `pure.json` deve ser criado como Secret no mesmo namespace onde o Portworx será instalado.

Nome obrigatório:

```text
px-pure-secret
```

Exemplo:

```bash
kubectl create secret generic px-pure-secret \
  --namespace portworx \
  --from-file=pure.json=pure.json
```

Validar:

```bash
oc get secret px-pure-secret -n portworx
```

---

# **Host Configuration no FlashArray**

## **Fibre Channel**

Criar hosts ou host group para os nodes OpenShift.

Registrar:

|**Host**|**WWPNs**|
|---|---|
|node01||
|node02||
|node03||

---

## **iSCSI**

Criar hosts ou host group para os nodes OpenShift.

Registrar:

|**Host**|**IQN**|
|---|---|
|node01||
|node02||
|node03||

---

# **Validação Pós-Configuração**

## **FlashArray**

Validar:

- Hosts criados
- WWPNs ou IQNs associados corretamente
- Conectividade ativa
- Volumes provisionados pelo Portworx aparecem no array

---

## **OpenShift / Portworx**

Validar:

```bash
pxctl status
```

```bash
pxctl cluster list
```

```bash
oc get pvc -A
```

Resultado esperado:

- Portworx Online
- Nodes Online
- PVCs provisionados com sucesso
- Volumes visíveis no FlashArray

---

# **Observações**

- Não armazenar credenciais reais neste documento.
- Não armazenar API Token real no Obsidian.
- Registrar somente placeholders ou referências seguras.
- Atualizar este documento após a primeira instalação real em POA.