# k8s-services
## Dependencies

Please prepare following packages:

- kubectl
- awscli
- helm
- helmfile
- flux2


### kubectl
At first, install kubectl, refer: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl .

### helm
I'm using helm to install cluster components, like kiam and alb-ingress-controller. So you have to install helm and helmfile. I use helm v3.x, so don't use helm v2.x.

- helm: https://helm.sh/docs/intro/install/
- helmfile: https://github.com/roboll/helmfile


### flux2
I'm using flux2 to deploy applications. Please install it: https://toolkit.fluxcd.io/get-started/

## Initialize
Please create kube config file like: `$HOME/.kuge/config-external-prd`

It is provided from kops: https://github.com/h3poteto/k8s-clusters

After create kube config, please set environment variables.

```bash
$ export KUBECONFIG=$HOME./kube/config-external-prd
$ kubectl get all
```

And you have to apply configmap for aws-iam-authenticator.

```bash
$ kubectl apply -f configmap-aws-auth.yml
```

## Create
### Namespace

```bash
$ kubectl apply -f namespace.yml
```
### helm packages
These packages are managed by helmfile, so you only run helmfile.

```bash
$ helmfile sync
```

After that, you can config it.

```bash
$ helm status aws-load-balancer-controller -n kube-system
NAME: aws-load-balancer-controller
LAST DEPLOYED: Fri Feb 26 21:13:03 2021
NAMESPACE: kube-system
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
AWS Load Balancer controller installed!
```
### deploy user

```bash
$ kubectl apply -f deploy-user-group.yml
```

## Deploy

I'm using flux2 to deploy applications.

Please setup deploy for the app. For example:

```bash
$ flux bootstrap github --owner=h3poteto --repository=k8s-services --path=external-prd/flux --private=false --branch=master
$ flux create source git k8s-services --url=https://github.com/h3poteto/k8s-services --branch=master --interval=1m
$ flux create kustomization pleromaio --source=k8s-services --path="./external-prd/pleromaio" --prune=true --interval=5m --validation=client
```

After you create kustomization, the application will be deployed automatically by flux2.
