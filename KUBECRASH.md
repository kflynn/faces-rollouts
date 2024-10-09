# Argo-Linkerd-Demo

<!-- @import demosh/check-requirements.sh -->
<!-- @import demosh/check-github.sh -->

### Prerequisites:

- A Kubernetes cluster of your choosing: for this demo we're using K3D running
  locally.

- The `kubectl`, `argocd`, `linkerd`, and `step` CLIs.

- A fork of this repo.

These requirements will be checked automatically if you use `demosh` to run
this.

<!-- @start_livecast -->
<!-- @SHOW -->

## Verify that your cluster can run Linkerd

Start by making sure that your Kubernetes cluster is capable of running
Linkerd.

```bash
linkerd check --pre
```

You should have all the checks pass. If there are any checks that do not pass,
make sure to follow the provided links in the output and fix those issues
before proceeding.

<!-- @wait_clear -->

## Install Argo CD in your cluster

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This will create a new namespace, `argocd`, where Argo CD services and
application resources will live, and install Argo CD itself. Wait for all the
Argo CD pods to be running:

```bash
kubectl -n argocd rollout status deploy,statefulset
```

Then use port-forwarding to access the Argo CD UI:

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443 > /dev/null 2>&1 &
```

The Argo CD UI should now be visible when you visit https://localhost:8080/.

<!-- @browser_then_terminal -->

To get the initial admin password, run:

```bash
argocd admin initial-password -n argocd
```

Then log into the UI with username as `admin` and the password from the output
above.

Note: This password is meant to be used to log into initially, after that it
is recommended that you delete the `argocd-initial-admin-secret` from the
`argocd` namespace once you have changed the password. You can change the
admin password with `argocd account update-password`. Since this is only for
demo purposes, we will not be showing this.

<!-- @browser_then_terminal -->

We'll be using the `argocd` CLI for our next steps, which means that we need to authenticate the CLI:

```bash
password=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)

argocd login 127.0.0.1:8080 \
  --username=admin \
  --password="$password" \
  --insecure
```

<!-- @wait_clear -->

## Add ArgoCD apps for Linkerd

We have to add three ArgoCD applications for Linkerd: one for Linkerd's CRDs,
one for Linkerd itself, and one for Linkerd Viz. But first: certificates.

### Generate certificates for Linkerd

In the Real World, you'd do this by having ArgoCD install cert-manager for
you, but we'll just do it by hand for the moment. Start by creating the
certificates using `step`:

Generate the trust anchor certificate:

```bash
#@immed
mkdir -p certs
#@immed
rm -f certs/ca.crt certs/ca.key
step certificate create \
     --profile root-ca \
     --no-password --insecure \
     root.linkerd.cluster.local \
     certs/ca.crt certs/ca.key
```

Generate the identity issuer certificate and key:

```bash
#@immed
rm -f certs/issuer.crt certs/issuer.key
step certificate create \
     --profile intermediate-ca --not-after 8760h \
     --ca certs/ca.crt --ca-key certs/ca.key \
     --no-password --insecure \
     identity.linkerd.cluster.local \
     certs/issuer.crt certs/issuer.key
```

<!-- @wait_clear -->

```bash,run
kubectl create namespace linkerd

kubectl create configmap \
        linkerd-identity-trust-roots -n linkerd \
        --from-file='ca-bundle.crt'=certs/ca.crt

kubectl create secret generic \
        linkerd-identity-issuer -n linkerd \
        --type=kubernetes.io/tls \
        --from-file=ca.crt=certs/ca.crt \
        --from-file=tls.crt=certs/issuer.crt \
        --from-file=tls.key=certs/issuer.key
```

Done.

<!-- @wait -->

### Add the `linkerd` applications

```bash
bat argocd/linkerd-crds.yaml
kubectl apply -f argocd/linkerd-crds.yaml
bat argocd/linkerd-control-plane.yaml
kubectl apply -f argocd/linkerd-control-plane.yaml
bat argocd/linkerd-viz.yaml
kubectl apply -f argocd/linkerd-viz.yaml
```

And now we should see our three Linkerd applications in the Argo dashboard.

<!-- @browser_then_terminal -->

```bash
argocd app sync linkerd
```
