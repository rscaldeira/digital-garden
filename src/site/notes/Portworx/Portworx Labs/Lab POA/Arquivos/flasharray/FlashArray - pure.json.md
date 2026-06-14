---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/flasharray/FlashArray - pure.json/","dg-note-properties":{}}
---

# FlashArray - pure.json

## Objetivo

Criar o arquivo `pure.json` com os dados do FlashArray para integração com Portworx.

## Quando usar

* Instalação do Portworx com FlashArray como backend
* Ambiente com um ou mais FlashArrays
* Cenários com ActiveCluster, quando necessário

## Exemplo básico

```json
{
  "FlashArrays": [
    {
      "MgmtEndPoint": "fa-mgmt.lab.local",
      "APIToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
  ]
}
```

## Exemplo com dois arrays

```json
{
  "FlashArrays": [
    {
      "MgmtEndPoint": "fa-a.lab.local",
      "APIToken": "xxxxxxxx-xxxx-xxxx-xxxx-aaaaaaaaaaaa"
    },
    {
      "MgmtEndPoint": "fa-b.lab.local",
      "APIToken": "xxxxxxxx-xxxx-xxxx-xxxx-bbbbbbbbbbbb"
    }
  ]
}
```

## Exemplo com realm

```json
{
  "FlashArrays": [
    {
      "MgmtEndPoint": "fa-mgmt.lab.local",
      "APIToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "Realm": "tenant-a"
    }
  ]
}
```

## Observações

* `MgmtEndPoint` deve ser IP ou FQDN, sem `https://`
* Para IPv6, usar colchetes
* `Realm` é usado somente em secure multi-tenancy
* Para FlashArray block, `NFSEndPoint` não é necessário
* Se você adicionar novos arrays ao `pure.json`, o Portworx descobre automaticamente depois de atualizar o secret

## Exemplo de IPv6

```json
{
  "FlashArrays": [
    {
      "MgmtEndPoint": "[2001:db8::10]",
      "APIToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
  ]
}
```
