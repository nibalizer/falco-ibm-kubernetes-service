Note: this does not work. I hope to return to it and get it working soon. I am leaving it up because it has some useful IAM shenanigans and because github doesn't let me press the 'make private' button on forked repos. The best way to get started with Falco and the IBM Cloud is here: https://developer.ibm.com/tutorials/installing-and-using-sysdig-falco/


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
#

## Provision IBM Event Streams and connect to falco events

TODO: create IES instance

### Get IES credentials

```bash
$ ibmcloud service list | grep falco
nibz-message-hub-falco-2                             messagehub                                    standard                                                                                                                                                                                                                                                                                                        create succeeded
$ ibmcloud service key-show  nibz-message-hub-falco-2 'Service credentials-1'
Invoking 'cf service-key nibz-message-hub-falco-2 Service credentials-1'...

Getting key Service credentials-1 for service instance nibz-message-hub-falco-2 as skrum@us.ibm.com...

{
 "api_key": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
 "instance_id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
 "kafka_admin_url": "https://kafka-admin-prod02.messagehub.services.us-south.bluemix.net:443",
 "kafka_brokers_sasl": [
  "kafka05-prod02.messagehub.services.us-south.bluemix.net:9093",
  "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093",
  "kafka01-prod02.messagehub.services.us-south.bluemix.net:9093",
  "kafka02-prod02.messagehub.services.us-south.bluemix.net:9093",
  "kafka03-prod02.messagehub.services.us-south.bluemix.net:9093"
 ],
 "kafka_rest_url": "https://kafka-rest-prod02.messagehub.services.us-south.bluemix.net:443",
 "mqlight_lookup_url": "https://mqlight-lookup-prod02.messagehub.services.us-south.bluemix.net/Lookup?serviceId=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
 "password": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
 "user": "xxxxxxxxxxxxxxxx"
}
```


### Create Kubernetes secret from IES credentials

```bash
ibmcloud service key-show  nibz-message-hub-falco-2 'Service credentials-1' | grep -v '\.\.\.' | jq '.' 
```

> Validate the downloaded credentials file looks correct.

```bash
ibmcloud service key-show  nibz-message-hub-falco-2 'Service credentials-1' | grep -v '\.\.\.' | jq '.' > kafka-credentials.json 

kubectl create secret generic eventstreams-binding  --from-file=binding=./kafka-credentials.json     
secret/eventstreams-binding created
```

> Validate that the secret looks correct on the Kubernetes side

```bash
kubectl get secret eventstreams-binding -o yaml | grep binding: | cut -d " " -f 4 | base64 -d | jq '.
```




### Setup authentication for openwhisk to kubernetes
#
```bash
 ibmcloud iam access-group-create falco-pod-deleter
Creating access group falco-pod-deleter under account IBM as skrum@us.ibm.com...
OK

                  
Name:          falco-pod-deleter   
ID:            AccessGroupId-937ee16b-5277-4143-9937-fc03ce19020c   
Description:  
```

service id
- add policy
- add api key


ibmcloud iam service-id-create falco-pod-deleter -d "Delete pods on falco events (nibz)"
Creating service ID falco-pod-deleter bound to current account as skrum@us.ibm.com...
OK
Service ID falco-pod-deleter is created successfully
                 
Name          falco-pod-deleter   
Description   Delete pods on falco events (nibz)   
CRN           crn:v1:bluemix:public:iam-identity::a/47b84451ab70b94737518f7640a9ee42::serviceid:ServiceId-0be7029f-a565-43c0-8a9a-2b8a41c352b8   
Bound To      crn:v1:bluemix:public:::a/47b84451ab70b94737518f7640a9ee42:::   
Version       1-41cd49c8b632191b5e9642c2b541bae9   
Locked        false   
UUID          ServiceId-0be7029f-a565-43c0-8a9a-2b8a41c352b8   


$ ibmcloud ks cluster-get nibz-istio-1.0-test-deleteme
Retrieving cluster nibz-istio-1.0-test-deleteme...
OK

                           
Name:                   nibz-istio-1.0-test-deleteme   
ID:                     9587cb6ec8684f52ba58bf785bbc9b7e   
State:                  normal   
Created:                2018-07-31T21:35:26+0000   
Location:               dal10   
Master URL:             https://169.47.70.10:31962   
Master Location:        Dallas   
Master Status:          Ready (2 weeks ago)   
Ingress Subdomain:      -   
Ingress Secret:         -   
Workers:                3   
Worker Zones:           dal10   
Version:                1.10.7_1520   
Owner Email:            skrum@us.ibm.com   
Monitoring Dashboard:   -   



$ ibmcloud iam service-policy-create falco-pod-deleter  --roles 'Administrator' --service-name containers-kubernetes --service-instance 9587cb6ec8684f52ba58bf785bbc9b7e --region 'us-south' 
Creating policy under current account for service ID falco-pod-deleter as skrum@us.ibm.com...
OK
Service policy is successfully created

                
Policy ID:   bfeb7768-4d9d-4749-ab9a-f32295c70c5c   
Version:     1-2e9de9f34d2a9fe5a58e9c39901fad64   
Roles:       Administrator   
Resources:                            
             Service Name       containers-kubernetes      
             Service Instance   9587cb6ec8684f52ba58bf785bbc9b7e      
             Region             us-south      
             Resource Type            
             Resource                 


$  ibmcloud iam service-api-key-create falco-pod-deleter falco-pod-deleter -d "API Key for OpenWhisk to delete pods" --file apikey 
Creating API key falco-pod-deleter of service falco-pod-deleter as skrum@us.ibm.com...
OK
Service API key falco-pod-deleter is created
Successfully save API key information to apikey

$ cat apikey 
{
        "name": "falco-pod-deleter",
        "description": "API Key for OpenWhisk to delete pods",
        "apikey": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "createdAt": "2018-09-27T01:40+0000",
        "locked": false,
        "uuid": ""
}


$ ibmcloud logout
Logging out...
OK


```
                






References:

https://www.slideshare.net/KnoxAnderson/falco-webinar
https://www.slideshare.net/MichaelDucy/container-runtime-security-with-falco  -- In particular slide 32
https://github.com/draios/falco-extras



