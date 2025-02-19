---
title: "Kubernetes Need to Know - API Server CA"
type: "Authentication"
venue: "K8S"
location: "Cloud Computing Matrix"
collection: k8s
category: Security
permalink: /k8s/2024-02-18-k8s-apiserver-ca
excerpt: 'Config Kubernetes API Server CA'
date: 2024-02-18
paperurl: 'http://deepnodes.github.io//k8s/2024-02-18-k8s-apiserver-ca.md'
citation: 'Deepnodes, Explore. (2024). &quot; Kubernetes Need to Know - CAs.&quot; <i>Cloud Computing Matrix</i>. 1(3).'
---


```sh
### ca is in kubelet.conf
$ cat /etc/kubernetes/kubelet.conf
	apiVersion: v1
	clusters:
	- cluster:
		certificate-authority-data: xxxxxxxxxxxxxx
		server: https://xxxx:6443
	name: kubernetes
	contexts:
	- context:
		cluster: kubernetes
		user: system:node:master
	name: system:node:master@kubernetes
	current-context: system:node:master@kubernetes
	kind: Config
	preferences: {}
	users:
	- name: system:node:master
	user:
		client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
		client-key: /var/lib/kubelet/pki/kubelet-client-current.pem

### decode, and save to file
$ echo xxxxxxxxxxx | base64 -d > /etc/kubernetes/pki/apiserver-kubelet-ca.crt
$ cat /etc/kubernetes/pki/apiserver-kubelet-ca.crt
	-----BEGIN CERTIFICATE-----
	yyyyyyyyyyyyyyy
	-----END CERTIFICATE-----

### set this ca file in the apiserver config file:
$ vim  /etc/kubernetes/manifests/kube-apiserver.yaml
	...
	--kubelet-certificate-authority=/etc/kubernetes/pki/apiserver-kubelet-ca.crt
	...
```
