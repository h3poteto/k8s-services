# k8s-services
## Dependencies

Please prepare following packages:

- kubectl
- flux2


### kubectl
At first, install kubectl, refer: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl .

### flux2
I'm using flux2 to deploy applications. Please install it: https://toolkit.fluxcd.io/get-started/

## Initialize
Please create kube config file like: `$HOME/.kuge/config-external`.

After create kube config, please set environment variables.

```bash
$ export KUBECONFIG=$HOME./kube/config-external-prd
$ kubectl get all
```

## Deploy

I'm using flux2 to deploy applications.

Please setup deploy for the app. For example:

```bash
$ flux bootstrap github --owner=h3poteto --repository=k8s-services --path=external/flux --private=false --branch=master
```

After you bootstrap flux, all resources will be synced automatically.
