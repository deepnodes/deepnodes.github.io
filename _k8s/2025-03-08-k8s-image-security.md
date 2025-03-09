---
title: "Kubernetes Security - Image Scanning"
type: "Security"
venue: "K8S"
location: "Cloud Computing Matrix"
collection: k8s
category: Security
permalink: /k8s/2025-03-08-k8s-image-security
excerpt: 'Image Scanning for Kubernetes Containers.'
date: 2025-03-08
paperurl: 'http://deepnodes.github.io//k8s/2025-03-08-k8s-image-security.md'
citation: 'Deepnodes, Explore. (2025). &quot; Kubernetes Security - Image Scanning.&quot; <i>Cloud Computing Matrix</i>. 1(3).'
---

### Supply Chain Security - Images Scanning

Perform static analysis of user workloads and container images.

E.g. Kubesec, KubeLinter, Trivy

### How to

**Use Trivy to Detect Vulnerable Images used by Pods**

Use the Trivy open-source container scanner to identify images with severe vulnerabilities used by Pods in a namespace.
Scan for images with High or Critical severity vulnerabilities, which should be removed.

```sh
$ trivy --version

# get images
$ kubectl get pods -o custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[*].image, RSRC:.metadata.resourceVersion" >> images.txt
  or kubectl get pods -o yaml | grep image:           # add ":"
# start to scan
$ for i in {alpine-v17.6.0,amazonlinux:1,amazonlinux:2,nginx:1.19,vicuu/nginx:host}; do trivy image --skip-db-update -s "HIGH,CRITICAL" $i >> images-scanned.txt; done

# Check the resault
cat images-scanned.txt | grep -i -B3 total

# Delete pods using vunarable images.
$ kubectl delete pod ** --force # 删除用到10.txt中的image的Pod
  or k scale deploy xxx --replicas 0
```

### Ref
Key words: trivy
