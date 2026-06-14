---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/storageclasses/Storage Classes Overview/","dg-note-properties":{}}
---


# Storage Classes Utilizadas

## Objetivo

Definir Storage Classes padrão para o Lab POA com foco em demonstrações de aplicações, OpenShift Virtualization e testes de resiliência.

---

## Apps / Containers

### px-app-repl1

Uso:

- Pods simples
- FIO
- Testes descartáveis
- Validações rápidas

Replica:

- 1

---

### px-app-repl2

Uso:

- Aplicações containerizadas
- HA básico
- Demonstrações simples de persistência

Replica:

- 2

---

### px-app-repl3

Uso:

- Demonstração de resiliência
- Failure scenarios
- Testes de indisponibilidade de node

Replica:

- 3

---

## OpenShift Virtualization

### px-vm-block-repl2

Uso:

- Storage default para VMs
- VMs Windows
- VMs Linux
- Discos C:
- Discos D:

Replica:

- 2

Observação:

Esta deve ser a Storage Class principal para máquinas virtuais.

---

### px-vm-block-repl1

Uso:

- VMs de lab sem HA
- Testes rápidos
- Economia de capacidade

Replica:

- 1

---

### px-rwx-file-kubevirt

Uso:

- Shared filesystem
- vTPM
- Casos específicos que exigem RWX

Replica:

- 2

Observação:

Não utilizar esta Storage Class para o disco principal da VM.

---

## CDI / Importação de Imagens

### px-cdi-scratch

Uso:

- Importação de imagens
- Boot sources
- CDI scratch space

Replica:

- 1

---

## Resumo

| Storage Class | Uso Principal |
|---|---|
| px-app-repl1 | Pods simples / FIO / testes descartáveis |
| px-app-repl2 | Apps containerizadas com HA básico |
| px-app-repl3 | Demo de resiliência / failure scenarios |
| px-vm-block-repl2 | Default para VMs |
| px-vm-block-repl1 | VMs de lab sem HA |
| px-rwx-file-kubevirt | Shared filesystem / vTPM |
| px-cdi-scratch | Importação / boot sources / CDI |