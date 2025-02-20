---
title: "Kubernetes Monitoring - Log Audit"
type: "Log Audit"
venue: "K8S"
location: "Cloud Computing Matrix"
collection: k8s
category: Monitoring
permalink: /k8s/2024-02-18-k8s-log-audit
excerpt: 'Kubernetes audit logs provide a detailed record of all activities that occur within a cluster.'
date: 2024-02-19
paperurl: 'http://deepnodes.github.io//k8s/2024-02-18-k8s-log-audit.md'
citation: 'Deepnodes, Explore. (2024). &quot; Kubernetes Monitoring - Log Audit.&quot; <i>Cloud Computing Matrix</i>. 1(3).'
---

### 0.Notes

[Log Audit](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)

Kubernetes audit logs provide a detailed record of all activities that occur within a cluster. These logs capture information about requests made to the Kubernetes API server, including who made the request, what actions were performed, and whether the requests were successful or denied.

**Configurable Parameters**
Will be mentioned in kube-apiserver.yaml
    - Where the logs are *stored*, e.g: "/var/log/kubernetes/kubernetes-logs.log"
    - How long will Log files be *retained*, e.g: 5 *days*.
    - How many logs will be retained, e.g: at maximum, a number of *10* old audit logs *files* are *retained*.

**Log Policy**
The base policy is located on the cluster’s master node.
  E.g, /etc/kubernetes/logpolicy/sample-policy.yaml
And this will be mentioned in kube-apiserver.yaml too.

Log policy can specifies what not to log, e.g:
- Do not log watch requests by the "system:kube-proxy" on endpoints or services
And it can specify what to log, e.g:
- Cronjobs persistentvolumes changes at RequestResponse level.
  记录 RequestResponse 级别的 persistentvolumes 相关的更改。
- Log namespaces changes at RequestResponse level
  记录 RequestResponse 级别的 namespaces 相关的更改。
- Log all other resources in core and extensions at the Request level.
  记录 Request level 的所有 core 和 extensions 的其他资源。
- Log all configMap and secret changes in all namespaces at the Metadata level
  Metadata 级别的所有 namespace 中的 ConfigMap 和 Secret 的更改。
- A catch-all rule to log all other requests at the Metadata level.
  全方位的规则以在 Metadata 级别记录所有其他请求。
- Log the request body of deployments/pod changes in a namespace.
  记录某个 namespace 中 configmaps/deployments 更改的 request body请求体。

**API Server Configure File's Location**
Master Node: /etc/kubernetes/manifests/kube-apiserver.yaml

### 1.How To

**Prepare Log Policy**

```sh
$ ssh to master node
### This location will be mentioned in kube-apiserver.yaml 
$ mkdir /etc/kubernetes/logpolicy/
$ vi /etc/kubernetes/logpolicy/policy.yaml
    ---
    apiVersion: audit.k8s.io/v1
    kind: Policy
    omitStages:
     - "RequestReceived"
    rules:
     # Do not log watch requests by the "system:kube-proxy" on endpoints or services
     - level: None         
    users: ["system:kube-proxy"]           
    verbs: ["watch"] 
    resources:
    - group: ""
      resources: ["endpoints", "services"]
      # Don't log authenticated requests to certain non-resource URL paths.
   - level: None                              
     userGroups: ["system:authenticated"]
     nonResourceURLs:
     - "api*"                                 # Wildcard matching.
     - "/version"

     # Log namespaces or persistentvolumes changes at RequestResponse level  
     - level: RequestResponse
       resources:
       - group: ""
         resources: ["namespaces","persistentvolumes"]
     # Log the request body of deployments/configmaps changes in the namespace kube-system.
     - level: Request 
       resources:
       - group: ""
         resources: ["configmaps"]
       namespaces: ["kube-system"]
     # Log all other resources in core and extensions at the Request level.
     - level: Request
       resources:
       - group: "" # core API group
       - group: "extensions" # Version of group should NOT be included.
     - level: Metadata
       resources:
       - group: ""
         resources: ["secrets","configmaps"]
     # A catch-all rule to log all other requests at the Metadata level.
     # Long-running requests like watches that fall under this rule will not generate an autit event in RequestReceived.
     - level: Metadata
       omitStages:
        - "RequestReceived"
```

**Enable Log Audit**

```sh
## where to store the logs
$ mkdir /var/log/kubernetes/

$ vim /etc/kubernetes/manifests/kube-apiserver.yaml 
# Search Log Audit or Auditing - doc-tasks - https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/
spec:
  containers:
  - args:
    - --audit-policy-file=/etc/kubernetes/logpolicy/policy.yaml       # add
    - --audit-log-path=/var/log/kubernetes/audit-logs.log      # add
    - --audit-log-maxage=5                                     # add
    - --audit-log-maxbackup=10 
  ......
    volumeMounts:
    - mountPath: /var/log/kubernetes/              # add
      name: log-dir                               # add
      readOnly: false
    - mountPath: /etc/kubernetes/logpolicy/policy.yaml      # add
      name: policy-file                                           # add
      readOnly: true
  ....
  volumes:
   - hostPath:
      path: /etc/kubernetes/logpolicy/policy.yaml
      type: File
    name: policy-file
    - name: log-dir
    hostPath:
      path: /var/log/kubernetes/
      type: DirectoryOrCreate
```

**Apply Log Policy**

```sh
# restart kubelet
systemctl daemon-reload
systemctl restart kubelet

#k get pods -n kube-system | grep api
cat /var/log/kubernetes/audit-logs.log 
tail -f /var/log/kubernetes/audit-log.log
```
