---
layout: post
title: "kubectl cheatsheet"
description: ""
category: CheatSheet
tags: [kubectl,kubernetes,cheatsheet]
group: post
fenced_code_blocks: true
comments: true
sharing: true
---

{% include JB/setup %}

# kubectl 的进阶用法
以下列出了一些使用 jsonpath 对 kubectl get 的结果进行定制化输出的操作

{% raw %}

* 查询 pod

```bash
kubectl get pods -o wide
```


* 查询 node 的 condition

```bash
kubectl get nodes -o jsonpath="{range .items[*]}{.status.conditions[?(@.status=='True')].type} {.metadata.name}{'\n'}{end}"
```

* 查看容器的日志


```bash
kubectl logs -f --tail=20 prometheus-k8s-0  -c prometheus
```

* 在容器内执行命令


```bash
kubectl exec -ti prometheus-k8s-0 -c prometheus -- bash
```

* 查找非 running 状态的 Pod


```bash
kubectl get pods -A --field-selector=status.phase!=Running | grep -v Complete
```

* 获取节点列表及其内存容量


```bash
kubectl get node -o jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.status.capacity.memory}{'\n'}{end}"
```

* 每个节点上调度的 Pod 个数


```bash
kubectl get pod --all-namespaces -o jsonpath="{range .items[*]}{.spec.nodeName}{'\n'}{end}"|sort|uniq -c
```

* 找出 DaemonSet 中未运行 pod 的节点


```bash
kubectl get node|grep -v "$(kubectl -n ${ns} get pod --all-namespaces -o wide | fgrep ${pod_template} | awk '{print $8}' | xargs -n 1 | sed 's/[[:space:]]*//g')"
```

* 按重启次数排序


```bash
kubectl get pods —sort-by=.status.containerStatuses[0].restartCount
```

* Pod 的 requests limits


```bash
kubectl get pods -A -o=custom-columns='NAME:spec.containers[*].name,MEMREQ:spec.containers[*].resources.requests.memory,MEMLIM:spec.containers[*].resources.limits.memory,CPUREQ:spec.containers[*].resources.requests.cpu,CPULIM:spec.containers[*].resources.limits.cpu'
```

* 从 ConfigMap 中创建文件


```bash
cm_name=config
ns=default
files=$(kubectl get cm $cm_name -n $ns -o go-template='{{range $k,$v := .data}}{{$k}}{{"\n"}}{{end}}')
for f in $(echo $files);do
  kubectl get cm $cm_name -n $ns -o go-template='{{range $k,$v := .data}}{{ if eq $k "'$f'"}}{{$v}}{{end}}{{end}}' > $f 2>/dev/null
done
```

* 一堆 Pod 中执行命令


```bash
appname=es
for p in `kubectl get po -l app=$appname -o name`;do
  echo $p
  kubectl exec ${p:4} -- sh -c 'hostname -i'
done
```

* 获取 Pod 状态


```bash
export jsonpath='{range .items[*]}{"\n"}{@.metadata.name}{range @.status.conditions[*]}{"\t"}{@.type}={@.status}{end}{end}' 
kubectl get po -ao jsonpath="$jsonpath" && echo
```

* PV Released


```bash
kubectl get pv -o jsonpath='{range.items[?(@.status.phase=="Released")]}{.metadata.name}{"\n"}{end}'
```

* Pod on node-01


```bash
kubectl get pod --all-namespaces -o jsonpath='{range .items[?(@.spec.nodeName=="node-01")]}{.metadata.name}{"\n"}{end}'
```

* 所有容器使用的镜像


```bash
kubectl get pods -o jsonpath="{range .items[*]}{.metadata.name}{','}{.spec.containers[*].image}{'\n'}{end}" --all-namespaces
```

{% endraw %}

