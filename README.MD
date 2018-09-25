## Sysdig Falco

Falco from Sysdig is an open source software tool that watches the Linux kernel for system calls and generates events when it sees calls that matches its rules set. Called a 'behavioral activity monitor', Falco is written by the excellent folks over at Sysdig.

Today, we'll install Falco onto an IBM Kubernetes Service cluster, build out some custom rules, then attach it to a pipeline with IBM Event Streams and IBM Functions to take action when a flagged event occurs.

This is a remix of the [falco-nats](https://github.com/sysdiglabs/falco-nats) example but uses IBM Event Streams in place of NATS and IBM Cloud Functions in place of kubeless.


## Installing Falco on the IBM Kubernetes Service


Create the service account for the falco system in kubernete's RBAC.

https://github.com/sysdiglabs/falco-nats


```bash
kubectl apply -f falco-account.yaml 
serviceaccount/falco-account created
clusterrole.rbac.authorization.k8s.io/falco-cluster-role created
clusterrolebinding.rbac.authorization.k8s.io/falco-cluster-role-binding created
```

Setup the falco config files in a config map

```bash
kubectl create configmap falco-config --from-file=falco-config/
configmap/falco-config created
```

Next, install falco via daemonset(link to daemonset reference material).

```bash
kubectl create -f falco-daemonset-configmap.yaml 
daemonset.extensions/falco created
```

Now, use two terminals to watch falco in action


Terminal 1: Start watching the Falco event stream

```bash
kubectl exec -it $(kubectl get pod -o name | grep -m1 falco | cut -d "/" -f 2) -- cat /var/run/falco/nats
```

Terminal 2: Pop a shell 

```bash
kubectl exec -it $(kubectl get pod -o name | grep -m1 falco | cut -d "/" -f 2) -- /bin/bash
```

Terminal 1: See the output

```bash
 kubectl exec -it $(kubectl get pod -o name | grep -m1 falco | cut -d "/" -f 2) -- cat /var/run/falco/nats
{"output":"04:44:14.629652552: Notice A shell was spawned in a container with an attached terminal (user=root k8s.pod=falco-d7ml2 container=fb91c1e33a1d shell=bash parent=<NA> cmdline=bash  terminal=34816) k8s.pod=falco-d7ml2 container=fb91c1e33a1d","priority":"Notice","rule":"Terminal shell in container","time":"2018-09-25T04:44:14.629652552Z", "output_fields": {"container.id":"fb91c1e33a1d","evt.time":1537850654629652552,"k8s.pod.name":"falco-d7ml2","proc.cmdline":"bash ","proc.name":"bash","proc.pname":null,"proc.tty":34816,"user.name":"root"}}
```

> Note that you can see lots of context including container id and pod name


## Writing a custom falco rule


TODO


## 









References:

https://www.slideshare.net/KnoxAnderson/falco-webinar
https://www.slideshare.net/MichaelDucy/container-runtime-security-with-falco  -- In particular slide 32
https://github.com/draios/falco-extras



