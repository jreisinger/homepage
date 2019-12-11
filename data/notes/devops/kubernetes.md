Basics
======

Kubernetes is the operating system for cloud-native applications.

Configuration
-------------

```bash
cat ~/.kube/config
# or
kubectl config view
```

Namespace
---------

* group of objects in a cluster
* similar to a filesystem folder
* see [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for more 

```bash
# your current namespace
kubectl config get-contexts # search for asterisk and see column NAMESPACE

# all namespaces in a cluster
kubectl get namespaces
```

Context
-------

* to change the default namespace more permanently
* to manage different clusters
* to manage different users

```bash
# list contexts
kubectl config get-contexts

# switch context
kubectl config use-context <context-name>
```

Objects
=======

Basic objects

![Basic objects](https://github.com/jreisinger/notes/raw/master/static/kubernetes.png)

* everything in Kubernetes is represented by a RESTful resource aka. a Kubernetes object ([resources vs objects](https://stackoverflow.com/questions/52309496/difference-between-kubernetes-objects-and-resources))
* each object exists at a unique HTTP path, e.g. `https://your-k8s.com/api/v1/namespaces/default/pods/my-pod`
* the `kubectl` makes requests to these URLs to access the objects
* `get` is conceptually similar to `ps`

```sh
# view Kubernetes objects/resources
kubectl get all [-l app=nginx] # all resources [with a label app=nginx]
kubectl get <type>             # all resources of given type
kubectl get <type> <object>    # specific resource

# details about an object
kubectl describe <type> <object>

# create, update objects
kubectl apply -f obj.yaml

# delete objects
kubectl delete -f obj.yaml  # no additional prompting!
kubectl delete <type> <object>
```

Pod
---

* atomic unit of work in Kubernetes cluster
* Pod = one or more containers working together symbiotically
* all containers in a Pod always land on the same machine
* once scheduled to a node, Pods don't move
* each container runs its own cgroup but they *share* network, hostname and filesystem
* like a logical host
* if you want to persist data across multiple instances of a Pod, you need to use `PersistentVolumes`

```sh
# Pod manifest - just a text-file representation of the Kubernetes API object
$ cat kuard-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

```sh
# Creating a Pod
kubectl apply -f kuard-pod.yaml
```

What should I put into a single pod?

* "Will these containers work correctly if they land on different machines?"
* should go into a Pod: web server + git scynhronizer - they communicate via filesystem
* should go into separate Pods: Wordpress + DB - they can communicate over net

Deployment
----------

* object of type controller
* manages replicasets/pods

One way to create a deployment:

```bash
kubectl create deployment quotes-prod --image=reisinge/quotes
kubectl scale deployment quotes-prod --replicas=3
kubectl label deployment quotes-prod ver=1 env=prod
```

Service
-------

* object that solves the service discovery problem (i.e. finding things in K8s cluster)
* a way to create a named label selector (see `kubectl get service -o wide`)
* a service is assigned a VIP called a *cluster IP* -> load balanced across all the
  pods identified by the selector
* good for identifying services inside a cluster

One way to create a service:

```bash
kubectl expose deployment quotes-prod --port=80 --target-port=5000
```

Ingress
-------

https://kubernetes.io/docs/concepts/services-networking/ingress/

Looking beyond the cluster
==========================

* exposing services outside of the cluster
* for HTTP or HTTPS use [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* for other ports use service of type NodePort or LoadBalancer

NodePort

* it enhances a service
* in addition to a cluster IP, a service gets a port (user defined or picked by
    the system)
* every node in the cluster forwards traffic to that port to the service
* if you can reach any node in the cluster you can get to the service
* this can be intergrated with HW/SW load balancers to expose the service even furher

Tips and tricks
===============

Useful output flags:

```sh
-o wide       # more details
-o json       # complete object in JSON format
-o yaml       # complete object in YAML format
--v=6         # verbosity
--no-headers
```

Clean up objecs:

```sh
kubectl delete deployments --all [--selector="app=myapp,env=dev"]
```

Create a proxy server between localhost and K8s API server:

```sh
kubectl proxy &                  # create proxy
curl localhost:8001/api/v1/pods  # get list of pods
```

Explain resource types:

```sh
kubectl explain svc
```

Get pods per node:

```
kubectl get pods -o json --all-namespaces | jq '.items |
  group_by(.spec.nodeName) | map({"nodeName": .[0].spec.nodeName,
  "count": length}) | sort_by(.count) | reverse'
```

Generate resource manifest:

```
kubectl run demo --image=cloudnatived/demo:hello --dry-run -o yaml
```

Show logs:

```sh
kubectl logs [-f] <pod>
kubectl exec -it <pod> -- bash  # or sh instead of bash
```

Copy files:

```sh
kubectl cp <pod>:/path/to/remote/file /path/to/local/file
```

Access Pod via port forwarding:

```sh
kubectl port-forward kuard 8080:8080  # tunnel: localhost -> k8s master -> k8s worker node
```

Run containers for troubleshooting:

```sh
kubectl run demo --image=cloudnatived/demo:hello --expose --port 8888 # pod to troubleshoot
kubectl run nslookup --image=busybox --rm -it --restart=Never --command -- nslookup demo
kubectl run wget --image=busybox --rm -it --restart=Never --command -- wget -qO- http://demo:8888
```

* `--command` -- command to run instead of container's default entrypoint

Resources
=========

* [Kubernetes: Up and Running](https://www.safaribooksonline.com/library/view/kubernetes-up-and/9781491935668/) (2017, 2019)
* Managing Kubernetes (2018)
* Cloud Native DevOps with Kubernetes (2019)
* [Getting Started with Kubernetes](https://www.safaribooksonline.com/videos/getting-started-with/9780135237823) (video, 2018)