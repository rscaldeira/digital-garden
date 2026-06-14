---
{"dg-publish":true,"permalink":"/Portworx/Portworx Labs/Lab POA/Arquivos/openshift/machineconfig/99-px-fa-mc-multipath-udev-master.yaml/","dg-note-properties":{}}
---

apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  creationTimestamp:
  labels:
    machineconfiguration.openshift.io/role: master
  name: 99-portworx-multipath-udev-mc-master
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - contents:
          source: data:text/plain;charset=utf-8;base64,ZGVmYXVsdHMgewogICAgdXNlcl9mcmllbmRseV9uYW1lcyBubwogICAgZW5hYmxlX2ZvcmVpZ24gIl4kIgogICAgcG9sbGluZ19pbnRlcnZhbCAgICAxMAogICAgZmluZF9tdWx0aXBhdGhzIHllcwp9CgpkZXZpY2VzIHsKICAgIGRldmljZSB7CiAgICAgICAgdmVuZG9yICAgICAgICAgICAgICAgICAgICAgICJOVk1FIgogICAgICAgIHByb2R1Y3QgICAgICAgICAgICAgICAgICAgICAiRXZlcnB1cmUgRmxhc2hBcnJheSIKICAgICAgICBwYXRoX3NlbGVjdG9yICAgICAgICAgICAgICAgInF1ZXVlLWxlbmd0aCAwIgogICAgICAgIHBhdGhfZ3JvdXBpbmdfcG9saWN5ICAgICAgICBncm91cF9ieV9wcmlvCiAgICAgICAgcHJpbyAgICAgICAgICAgICAgICAgICAgICAgIGFuYQogICAgICAgIGZhaWxiYWNrICAgICAgICAgICAgICAgICAgICBpbW1lZGlhdGUKICAgICAgICBmYXN0X2lvX2ZhaWxfdG1vICAgICAgICAgICAgMTAKICAgICAgICB1c2VyX2ZyaWVuZGx5X25hbWVzICAgICAgICAgbm8KICAgICAgICBub19wYXRoX3JldHJ5ICAgICAgICAgICAgICAgMAogICAgICAgIGZlYXR1cmVzICAgICAgICAgICAgICAgICAgICAwCiAgICAgICAgZGV2X2xvc3NfdG1vICAgICAgICAgICAgICAgIDYwCiAgICB9CiAgICBkZXZpY2UgewogICAgICAgIHZlbmRvciAgICAgICAgICAgICAgICAgICAiUFVSRSIKICAgICAgICBwcm9kdWN0ICAgICAgICAgICAgICAgICAgIkZsYXNoQXJyYXkiCiAgICAgICAgcGF0aF9zZWxlY3RvciAgICAgICAgICAgICJzZXJ2aWNlLXRpbWUgMCIKICAgICAgICBoYXJkd2FyZV9oYW5kbGVyICAgICAgICAgIjEgYWx1YSIKICAgICAgICBwYXRoX2dyb3VwaW5nX3BvbGljeSAgICAgZ3JvdXBfYnlfcHJpbwogICAgICAgIHByaW8gICAgICAgICAgICAgICAgICAgICBhbHVhCiAgICAgICAgZmFpbGJhY2sgICAgICAgICAgICAgICAgIGltbWVkaWF0ZQogICAgICAgIHBhdGhfY2hlY2tlciAgICAgICAgICAgICB0dXIKICAgICAgICBmYXN0X2lvX2ZhaWxfdG1vICAgICAgICAgMTAKICAgICAgICB1c2VyX2ZyaWVuZGx5X25hbWVzICAgICAgbm8KICAgICAgICBub19wYXRoX3JldHJ5ICAgICAgICAgICAgMAogICAgICAgIGZlYXR1cmVzICAgICAgICAgICAgICAgICAwCiAgICAgICAgZGV2X2xvc3NfdG1vICAgICAgICAgICAgIDYwMAogICAgfQp9CgpibGFja2xpc3RfZXhjZXB0aW9ucyB7CiAgICAgICAgcHJvcGVydHkgIihTQ1NJX0lERU5UX3xJRF9XV04pIgp9CgpibGFja2xpc3QgewogICAgICBkZXZub2RlICJecHhkWzAtOV0qIgogICAgICBkZXZub2RlICJecHhkKiIKICAgICAgZGV2aWNlIHsKICAgICAgICB2ZW5kb3IgIlZNd2FyZSIKICAgICAgICBwcm9kdWN0ICJWaXJ0dWFsIGRpc2siCiAgICAgIH0KfQo=
        filesystem: root
        mode: 0644
        overwrite: true
        path: /etc/multipath.conf
      - contents:
          source: data:text/plain;charset=utf-8;base64,QUNUSU9OPT0iY2hhbmdlIiwgU1VCU1lTVEVNPT0ic2NzaSIsIEVOVntTREVWX1VBfT09IkNBUEFDSVRZX0RBVEFfSEFTX0NIQU5HRUQiLCBURVNUPT0icmVzY2FuIiwgQVRUUntyZXNjYW59PSJ4IgpBQ1RJT049PSJjaGFuZ2UiLCBTVUJTWVNURU09PSJzY3NpIiwgRU5We1NERVZfVUF9PT0iQVNZTU1FVFJJQ19BQ0NFU1NfU1RBVEVfQ0hBTkdFRCIsIFRFU1Q9PSJyZXNjYW4iLCBBVFRSe3Jlc2Nhbn09IngiCkFDVElPTj09ImNoYW5nZSIsIFNVQlNZU1RFTT09InNjc2kiLCBFTlZ7U0RFVl9VQX09PSJSRVBPUlRFRF9MVU5TX0RBVEFfSEFTX0NIQU5HRUQiLCBSVU4rPSJzY2FuLXNjc2ktdGFyZ2V0ICRlbnZ7REVWUEFUSH0iCg==
        filesystem: root
        mode: 0644
        overwrite: true
        path: /etc/udev/rules.d/99-pure-storage.rules
    systemd:
      units:
      - enabled: true
        name: iscsid.service
      - enabled: true
        name: multipathd.service
