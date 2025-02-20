---
title: "Kubernetes Security - Analyze a Dockerfile"
type: "Security"
venue: "K8S"
location: "Cloud Computing Matrix"
collection: k8s
category: Security
permalink: /k8s/2024-02-18-k8s-docker-check
excerpt: 'Analyze a Dockerfile and fix instructions present in the file being prominent security best practice issues.'
date: 2024-02-19
paperurl: 'http://deepnodes.github.io//k8s/2024-02-19-k8s-docker-check.md'
citation: 'Deepnodes, Explore. (2024). &quot; Kubernetes Security - Analyze a Dockerfile.&quot; <i>Cloud Computing Matrix</i>. 1(3).'
---

## 0.Note

Key words：Dockerfile security issues

## 1.kubesec
```sh
docker run -i kubesec/kubesec:512c5e0 scan /dev/stdin < kubesec-test.yaml
```

## 2.Example

**Sample Docker file**

```docker
	FROM ubuntu: latest
	RUN apt-get update -y
	RUN apt-install nginx -y
	COPY entrypoint.sh /
	ENTRYPOINT ["/entrypoint.sh"]
	USER ROOT
```
Fix:
  FROM ubuntu: latest -> FROM ubuntu: 22.04
  USER ROOT -> USER <APPUSER>

**Sample YAML file**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: SAMPLE
spec:
  securityContext:
    runAsUser: 1000                                      # 1000 ->  6688
  containers:
  - name: demo 
    image: gcr.io/google-samples/node-hello:3.0
    securityContext: 
      runAsUser: 0                                       # 6688  
      privileged: True                                   # true -> false
      allowPrivilegeEscalation: false
    readOnlyRootFilesystem: false                        # false -> true

# label: run -> app
```
