# kind-examples

With the aim of using a [Kubernetes in Docker (kind)](https://kind.sigs.k8s.io/docs/user/quick-start/) cluster for local development, this has some examples and notes.


Set up a kind cluster exposing container ports using the `kind.config.yaml` which exposes the worker node on port 80 & 443.

```
kind create cluster --config kind.config.yaml
```

Note, although normally kind can create multiple worker nodes, when exposing ports via the kind.config.yaml, only one worker can be used, as Docker can only route to one container.

```bash
$ kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   11m   v1.16.1
kind-worker          Ready    <none>   10m   v1.16.1
```


# Contour

Contour uses Envoy to expose kubernetes externally.

Check out contour and apply everything in examples/contour

```
$ git clone https://github.com/projectcontour/contour
$ kubectl apply -f contour/examples/contour
namespace/projectcontour created
serviceaccount/contour created
configmap/contour created
customresourcedefinition.apiextensions.k8s.io/ingressroutes.contour.heptio.com created
customresourcedefinition.apiextensions.k8s.io/tlscertificatedelegations.contour.heptio.com created
customresourcedefinition.apiextensions.k8s.io/httpproxies.projectcontour.io created
customresourcedefinition.apiextensions.k8s.io/tlscertificatedelegations.projectcontour.io created
serviceaccount/contour-certgen created
rolebinding.rbac.authorization.k8s.io/contour created
role.rbac.authorization.k8s.io/contour-certgen created
job.batch/contour-certgen created
clusterrolebinding.rbac.authorization.k8s.io/contour created
clusterrole.rbac.authorization.k8s.io/contour created
role.rbac.authorization.k8s.io/contour-leaderelection created
rolebinding.rbac.authorization.k8s.io/contour-leaderelection created
service/contour created
service/envoy created
deployment.apps/contour created
daemonset.apps/envoy created
```

Verify all resources are up

```
kubectl -n projectcontour get pods,ds,deployments,services
```

Now you can add your own resources and front them with contour using contour's CRD HTTPProxy

See kuard-httproxy.yaml as an example of a deployment of kuard with an HTTPProxy.

```bash
kubectl apply -f kuard-httproxy.yaml
```

Monitor via:

```bash
kubectl get pods,svc,httpproxy -l app=kuard
```

and then go to http://localhost:80 in a browser to see kuard.

Another good example that shows of using all the aspects of the Contour HTTPProxy, see the article ["HTTPProxy in Action"](https://projectcontour.io/httpproxy-in-action/) on the projectcontour.io blog.