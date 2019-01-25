# k8s-services
## Dependencies

Please prepare following packages:

- kubect
- aws-iam-authenticator

At first, install kubectl, refer: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl .
Then install aws-iam-authenticator.

```bash
$ go get -u -v github.com/kubernetes-sigs/aws-iam-authenticator/cmd/aws-iam-authenticator
```


## Initialize
Please paste the output of https://github.com/h3poteto/h3poteto-terraform-aws/blob/master/clusters/base/prd/ap-northeast-1/output.tf#L2 to `~/.kube/config-h3poteto` .

And paste the output of https://github.com/h3poteto/h3poteto-terraform-aws/blob/master/clusters/base/prd/ap-northeast-1/output.tf#L6 to `~/.kube/node-h3poteto` .

```bash
$ export KUBECONFIG=$HOME./kube/config-h3poteto
$ kubectl get all
...

$ kubectl apply -f ~/.kube/node-h3poteto
```

## Create
### Namespace

```bash
$ kubectl -f namespace.yml
```

### kube2iam

```bash
$ kubectl -f kube2iam-sa.yml
$ kubectl -f kube2iam-ds.yml
```

After that:

```bash
$ kubectl get daemonsets -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
aws-node     1         1         1       1            1           <none>          1d
kube-proxy   1         1         1       1            1           <none>          1d
kube2iam     1         1         1       1            1           <none>          1h

$ kubectl get pods -n kube-system
NAME                       READY   STATUS    RESTARTS   AGE
aws-node-fxk9g             1/1     Running   0          4h
coredns-7774b7957b-jrtc9   1/1     Running   0          4h
coredns-7774b7957b-pq2kn   1/1     Running   0          4h
kube-proxy-mgqb8           1/1     Running   0          4h
kube2iam-ljs4s             1/1     Running   0          1h
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
