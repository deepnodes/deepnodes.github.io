---
title: "Kubernetes Security - Enable AppArmor on a K8S node"
type: "Security"
venue: "K8S"
location: "Cloud Computing Matrix"
collection: k8s
category: Security
permalink: /k8s/2025-03-08-k8s-system-hardening
excerpt: 'Enable AppArmor to harden the OS.'
date: 2025-03-08
paperurl: 'http://deepnodes.github.io//k8s/2025-03-08-k8s-system-hardening.md'
citation: 'Deepnodes, Explore. (2025). &quot; Kubernetes Security - Enable AppArmor.&quot; <i>Cloud Computing Matrix</i>. 1(3).'
---

### System Hardening - Enable AppArmor
AppArmor is a Linux kernel security module that can be used to restrict the capabilities of processes running on the host operating system. 
Each process can have its own security **profile**.
  Location: **/etc/apparmor.d**

[apparmor](https://kubernetes.io/docs/tutorials/clusters/apparmor/#%E4%B8%BE%E4%BE%8B)

### How to Enforce a Prepared AppArmor *profile*

```sh
$ ssh node-001
$ sudo -i

$ cat /sys/module/apparmor/parameters/enabled
$ cat /sys/kernel/security/apparmor/profiles | sort
    
$ cd /etc/apparmor.d
# prepare the AppArmor 
$ vi k8s-apparmor-example
      #include <tunables/global>

      profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
        #include <abstractions/base>

        file,

        # Deny all file writes.
        deny /** w,
      }

# check the AppArmor status
$ apparmor_status | grep k8s-apparmor-example-deny-write # 没有grep到说明没有启动
$ apparmor_parser -q k8s-apparmor-example-deny-write                # 加载启用这个配置文件
$ aa-status | grep k8s-apparmor-example-deny-write

$ exit
```


### How to Apply the AppArmor in a Pod
Next, run a simple "Hello AppArmor" Pod with the newly created and parsed profile:
```sh
(exit from the node)
$ vi aa-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-apparmor-example-deny-write
  containers:
  - name: hello
    image: busybox:1.28
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
  nodeName: node-001

#### or:
apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
  annotations:
    container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write   #"hello" is the container's name
spec:
  containers:
  - name: hello
    image: busybox
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
  nodeName: node-001                                                                       # the node which parsed the aa profile

$ kubectl apply -f aa-pod.yaml
$ k get pods

$ kubectl exec hello-apparmor -- cat /proc/1/attr/current

```
