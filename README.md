# kube-artifacts
This repo is for defining any configuration files used for our Kubernetes cluster and how to manually deploy to the cluster.

# Getting started
## Install kubectl
1. Install kubectl using (these instructions)[https://kubernetes.io/docs/tasks/tools/install-kubectl/]
2. Check that it is installed
```
$ kubectl version
```
and you should see something like:
```
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.1", GitCommit:"7879fc12a63337efff607952a323df90cdc7a335", GitTreeState:"clean", BuildDate:"2020-04-08T17:30:47Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```
## Allow kubectl access to the cluster
1. ssh into master node and copy `/etc/kubernetes/admin.conf` into `~/.kube/config` on your local machine
```
$ ssh bront@192.168.2.8 
<enter password>
$ cat /etc/kubernetes/admin.conf
```
2. Validate that you can access the kubernetes api
```
$ kubectl cluster-info
```
and you should get something like this:
```
Kubernetes master is running at https://192.168.2.8:6443
KubeDNS is running at https://192.168.2.8:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

Congrats! Now you can interact with the cluster from your local machine!

## Create a deployment
Defined in [deployment.yaml](./deployment.yaml) is the Deployment object we are going to create on our cluster. We specify a couple of things in this file:

1. The name of the deployment in `metadata.name`
2. How many pods via the `spec.replicas` property
3. The pod specifications in `spec.template` that specifies our container image and ports

After making sure that our `deployment.yaml` file looks correct, we apply it onto our cluster:
```
$ kubectl apply -f deployment.yaml
```

You should now be able to see your new deployment:
```
$ kubectl get deployments
$ kubectl describe deployment <deployment name>
```

To look at your pods you can use:
```
$ kubectl get pods
$ kubectl get pods -o wide
```

Using the `-o wide` lets you see more information on the pods like the IP and what node they're on. To view more detailed information you can do
```
$ kubectl describe pod <pod name>
```

## Create a service to expose the application on NodePort
Currently the app can only be accessed from inside the cluster (if you were to ssh into the master or worker nodes). How do we access it from our local machine? One way is to use a `Service` object to route the pod's IP to each of the Nodes.

```
$ kubectl expose deployment <DEPLOYMENT NAME> --type=NodePort --name=<SERVICE NAME>
```

Example:
```
$ kubectl expose deployment brontchat --type=NodePort --name=bront-service
```

This creates a Service that creates a random port from 30000-32767 on the nodes of the cluster. If you wish to specify a port you can do

```
$ kubectl expose deployment <DEPLOYMENT NAME> --type=NodePort --name=<SERVICE NAME> --port=<PORT ON NODE> --target-port=<PORT ON CONTAINER>
```

Verify that you created the service properly
```
$ kubectl get services
$ kubectl describe service <SERVICE NAME>
```

Now you should be able to hit the application by using `NODE_IP:NODE_PORT`