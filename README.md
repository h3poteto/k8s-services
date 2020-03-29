# k8s-services
## Dependencies

Please prepare following packages:

- kubectl
- awscli
- helm
- helmfile


### kubectl
At first, install kubectl, refer: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl .

### helm
I'm using helm to install cluster components, like kiam and alb-ingress-controller. So you have to install helm and helmfile. I use helm v3.x, so don't use helm v2.x.

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
### kiam, alb-ingress-controller, ...
These packages are managed by helmfile, so you only run helmfile.

```bash
$ helmfile sync
```

After that, you can config it.

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
