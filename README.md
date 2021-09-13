# racherinstall
install racher with helm

## create cert-manager for rancher

get cert-manager with helm repo
``` bash
$ kubectl create namespace ${namespace}
$ helm repo add jetstack https://charts.jetstack.io
$ helm repo update
```

``` bash
# Helm v3+
$ helm install \
  cert-manager jetstack/cert-manager \
  --namespace ${namespace} \
  --version v1.3.1 \
  --set installCRDs=true
```

``` bash
$ kubectl get pods --namespace cert-manager
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```

test issuer work

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  dnsNames:
    - example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
```

create own issuer or clusterissuer to ${namespace} for rancher certificate

``` yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: ${namespace} #if use issuer need namespace
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: ${ur mail}
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

## create rancher

``` bash
$ helm repo add rancher https://releases.rancher.com/server-charts/stable
$ helm repo update
```

install rancher

``` bash
$ helm install rancher rancher-latest/rancher \
  --namespace example \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```

-------------------------------------

``` bash
$ helm install rancher rancher-latest/rancher \
  --namespace example \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret's name
```

``` bash
$ kubectl get all -n cert-manager
NAME                                           READY   STATUS    RESTARTS   AGE
pod/rancher-9755f9bc8-75gv7                    1/1     Running   0          74m
pod/rancher-9755f9bc8-gpk2m                    1/1     Running   0          72m
pod/rancher-9755f9bc8-l5rls                    1/1     Running   0          74m

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/rancher                ClusterIP   xxx.xxx.xxx.xxx   <none>        80/TCP,443/TCP   74m

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rancher                   3/3     3            3           74m

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/rancher-9755f9bc8                    3         3         3       74m

```

210511 done.
