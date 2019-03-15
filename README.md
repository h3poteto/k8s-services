# k8s-services
## Dependencies

Please prepare following packages:

- kubectl
- aws-iam-authenticator
- helm
- helmfile


### kubectl
At first, install kubectl, refer: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl .

### aws-iam-authenticator
Then install aws-iam-authenticator.

```bash
$ go get -u -v github.com/kubernetes-sigs/aws-iam-authenticator/cmd/aws-iam-authenticator
$ which aws-iam-authenticator
```

### helm
I'm using helm to install cluster components, like kube2iam and alb-ingress-controller. So you have to install helm and helmfile.

- helm: https://helm.sh/docs/using_helm/#installing-helm
- helmfile: https://github.com/roboll/helmfile


## Initialize
Please create kube config file like: `$HOME/.kuge/config-external-prd`

It is provided from kops: https://github.com/h3poteto/k8s-clusters

After create kube config, please set environment variables.

```bash
$ export KUBECONFIG=$HOME./kube/config-external-prd
$ kubectl get all
```

## Create
### Namespace

```bash
$ kubectl apply -f namespace.yml
```

### aws-auth

```bash
$ kubectl apply -f configmap-aws-auth.yml
```

### helm
Please install helm to your cluster.

```bash
$ kubectl apply -f helm-sa.yaml
$ helm init --service-account tiller
$ helm version
Client: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
```

### kube2iam, alb-ingress-controller
These packages are managed by helmfile, so you only run helmfile.

```bash
$ helmfile sync
```

After that, you can config it.

```bash
$ helm status kube2iam
LAST DEPLOYED: Fri Mar 15 21:23:49 2019
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME            READY  STATUS   RESTARTS  AGE
kube2iam-htbjh  1/1    Running  0         68m
kube2iam-zbhm7  1/1    Running  0         68m

==> v1/ServiceAccount
NAME      SECRETS  AGE
kube2iam  1        68m
...
```

```bash
$ helm status alb-ingress-controller
LAST DEPLOYED: Fri Mar 15 21:23:49 2019
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                               AGE
alb-ingress-controller-aws-alb-ingress-controller  63m

==> v1/ClusterRoleBinding
NAME                                               AGE
alb-ingress-controller-aws-alb-ingress-controller  63m

==> v1/Deployment
NAME                                               READY  UP-TO-DATE  AVAILABLE  AGE
alb-ingress-controller-aws-alb-ingress-controller  1/1    1           1          63m
...
```

And you can see pods like this.

```bash
$ kubectl get pods -n kube-system
NAME                                                                    READY   STATUS    RESTARTS   AGE
alb-ingress-controller-aws-alb-ingress-controller-6cc74979jvfwx         1/1     Running   0          57m
aws-iam-authenticator-q9htn                                             1/1     Running   0          22d
aws-node-kscdp                                                          1/1     Running   0          22d
aws-node-nfjg8                                                          1/1     Running   1          11d
aws-node-xmrbt                                                          1/1     Running   1          11d
dns-controller-547884bc7f-vqh5l                                         1/1     Running   1          22d
etcd-server-events-ip-10-0-4-205.ap-northeast-1.compute.internal        1/1     Running   0          22d
etcd-server-ip-10-0-4-205.ap-northeast-1.compute.internal               1/1     Running   0          22d
kube-apiserver-ip-10-0-4-205.ap-northeast-1.compute.internal            1/1     Running   5          22d
kube-controller-manager-ip-10-0-4-205.ap-northeast-1.compute.internal   1/1     Running   0          22d
kube-dns-6b4f4b544c-rgm8n                                               3/3     Running   0          11d
kube-dns-6b4f4b544c-scvsl                                               3/3     Running   0          11d
kube-dns-autoscaler-6b658bd4d5-rth6n                                    1/1     Running   0          11d
kube-proxy-ip-10-0-33-117.ap-northeast-1.compute.internal               1/1     Running   0          11d
kube-proxy-ip-10-0-4-205.ap-northeast-1.compute.internal                1/1     Running   0          22d
kube-proxy-ip-10-0-9-22.ap-northeast-1.compute.internal                 1/1     Running   0          11d
kube-scheduler-ip-10-0-4-205.ap-northeast-1.compute.internal            1/1     Running   0          22d
kube2iam-htbjh                                                          1/1     Running   0          1h
kube2iam-zbhm7                                                          1/1     Running   0          1h
tiller-deploy-57c574bfb8-l2rzl                                          1/1     Running   0          2h
```

### ALB and Service


```bash
$ kubectl apply -f fascia/deployment.yml
$ kubectl apply -f fascia/service.yml
$ kubectl apply -f ingress.yml
```

### deploy user

```bash
$ kubectl apply -f deploy-user-group.yml
```

## Deploy

`deployment.yml` does not contain ECR repository and tag. So please update the repository, and deploy it.

```bash
$ kubectl apply -f fascia/deployment.yml
```

Or update command.

```bash
$ kubectl patch -f fascia/deployment.yml -p '{"spec":{"template":{"spec":{"containers":[{"name":"go","image":"123456789.dkr.ecr.ap-northeast-1.amazonaws.com/h3poteto/fascia:6ffcc35ff056cd7d97cc361efb8e663865663393"}]}}}}'
```
