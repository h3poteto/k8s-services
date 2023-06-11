# k8s-services
## Dependencies

Please prepare following packages:

- kubectl
- helmfile
- flux2


### kubectl
At first, install kubectl, refer: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl .

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


## Initial helm packages
These packages are managed by helmfile, so you only run helmfile.

```bash
$ helmfile sync
```

## Deploy

I'm using flux2 to deploy applications.

Please setup deploy for the app. For example:

```bash
$ flux bootstrap github --owner=h3poteto --repository=k8s-services --path=external-prd/flux --private=false --branch=master
```

After you bootstrap flux, all resources will be synced automatically.
