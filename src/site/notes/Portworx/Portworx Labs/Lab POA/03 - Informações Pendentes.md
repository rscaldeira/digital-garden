---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/03 - Informações Pendentes/","dg-note-properties":{}}
---



# **Informações Pendentes**

Este documento concentra apenas os itens que ainda precisam ser confirmados antes da implantação do Lab POA.

Itens já definidos foram removidos e documentados nos demais documentos do projeto.

---

# **Hardware**

## **Servidores**

Pendências:

- Modelo exato dos servidores
- Fabricante
- Modelo das CPUs
- Quantidade de CPUs por servidor
- Quantidade de memória RAM por servidor
- Quantidade de NICs por servidor
- Velocidade das interfaces de rede

## **Discos locais**

Pendências:

- Confirmar se haverá SSD local dedicado para metadata do PX-StoreV2
- Identificar device names dos SSDs locais
- Confirmar capacidade dos SSDs

Observação:

Caso o desenho final utilize metadata provisionada pelo FlashArray, os SSDs locais poderão ser dispensados.

---

# **Rede**

## **Endereçamento**

Pendências:

- Faixa IP definitiva do laboratório
- Gateway
- DNS
- VLAN utilizada pelo ambiente

## **MetalLB**

Pendências:

- Range definitivo do IPAddressPool
- Reserva formal dos IPs utilizados pelo MetalLB

Exemplo:

```text
192.168.x.x - 192.168.x.x
```

---

# **FlashArray**

## **Conectividade**

Pendências:

- Protocolo definitivo:
    - Fibre Channel
    - iSCSI
    - NVMe/TCP
    - NVMe/RDMA
- Endpoints de management
- Credenciais definitivas para criação do px-pure-secret

## **Integração Portworx**

Pendências:

- Definir estratégia final de metadata:
    - SSD local
    - Metadata provisionada pelo FlashArray
- Confirmar tamanho dos Cloud Drives utilizados pelo PX-StoreV2

---

# **OpenShift**

## **Instalação**

Pendências:

- Base Domain definitivo
- API VIP
- Ingress VIP

## **Acesso**

Pendências:

- Pull Secret utilizado na instalação
- Conta Red Hat utilizada para o deploy

---

# **Portworx**

## **StorageCluster**

Pendências:

- Gerar StorageCluster definitivo no Portworx Central Spec Generator
- Confirmar parâmetros finais:
    - SAN Type
    - Pool Size
    - Metadata Device
    - Journal Configuration
    - Número de Storage Nodes

---

# **OpenShift Virtualization**

## **Live Migration**

Pendências:

- Validar comportamento da StorageClass px-vm-block-repl2
- Executar teste real de Live Migration
- Validar comportamento durante Node Drain

## **Templates**

Pendências:

- Validar Fedora VM
- Validar Windows VM
- Validar CDI Import

---

# **PX-Backup**

## **Infraestrutura**

Pendências:

- Confirmar suporte AVX nos processadores do laboratório

Validação:

```bash
lscpu | grep -i avx
```

## **Instalação**

Pendências:

- Definir namespace definitivo
- Definir StorageClass definitiva
- Aplicar licença após instalação

---

# **Validação Final do Lab**

Itens que ainda precisam ser executados:

- Instalação do OpenShift
- Instalação do MetalLB
- Instalação do Portworx
- Aplicação das StorageClasses
- Instalação do OpenShift Virtualization
- Criação de Fedora VM
- Criação de Windows VM
- Testes de Snapshot
- Testes de Clone
- Testes de Live Migration
- Testes de Node Drain
- Instalação do PX-Central
- Instalação do PX-Backup
- Testes de Backup e Restore

---

# **Status Atual**

Fase atual:

- Documentação praticamente concluída
- Aguardando execução da implantação do ambiente
- Aguardando validação funcional dos componentes