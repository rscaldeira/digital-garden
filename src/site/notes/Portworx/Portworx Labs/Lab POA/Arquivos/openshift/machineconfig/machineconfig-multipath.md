---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/openshift/machineconfig/machineconfig-multipath/","dg-note-properties":{}}
---

# MachineConfig Multipath

Objetivo:

Configurar multipath para FlashArray.

Aplicação:

oc apply -f machineconfig-multipath.yaml

Validação:

multipath -ll