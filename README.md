# Faces Demo, GitOps Edition

This repo contains the application configuration for the [Faces demo],
which has been used for a great many things at this point, notably many
[Buoyant] [Service Mesh Academy] workshop and [KubeCrash] events.

## Using The GitOps Edition

This repo is intended to be used as part of a GitOps workflow; it was created
specifically for the [Real World GitOps with Argo CD and Linkerd] [Service
Mesh Academy] workshop. You can find that workshop source in the
[real-world-argo-linkerd] repo, and going through that workshop is the best
way to use this repo.

### Argo CD

Point Argo to the `argocd/applications` directory of this repo. The assumption
is that you've already installed Linkerd and Emissary-ingress into the
cluster.

### Other

Alternately, you can also just fork this repo and play around with whatever
GitOps setup you have going. You'll need a Kubernetes cluster into which
you've already installed [Linkerd], [Emissary-ingress], and the [Faces demo],
and both Emissary and Faces must be part of the Linkerd mesh.

You can look at the `argocd` directory for reference, but here are the steps:

1. Create the `faces` namespace. Make sure it's annotated with
   `linkerd.io/inject: enabled` (there's a suitable manifest in
   `argocd/resources/bootstrap/faces-namespace.yaml`).

2. Install the Faces Helm chart from
   `oci://registry-1.docker.io/dwflynn/faces-chart:0.8.0`.

3. Install the manifests in the `k8s` directory to configure Emissary to talk
   to Faces.

The end result should be that if you point your browser at the
`emissary-ingress` service in the `emissary` namespace, you'll see

- The Linkerd Viz dashboard at `/`
- The Faces demo at `/faces/`
- The `face` workload at `/face/` -- this is necessary for the Faces demo to
  work! but you won't be pointing a browser directly to it.

**Note**: The configuration here does _not_ configure anything fancy: no
retries, timeouts, circuit breaking, or any of that stuff. It's just the bare
minimum to get the demo working, to give you a place to stand to play with
using GitOps to configure whatever you want.

[Faces demo]: https://github.com/BuoyantIO/faces-demo
[Buoyant]: https://buoyant.io
[Service Mesh Academy]: https://buoyant.io/service-mesh-academy/
[KubeCrash]: https://kubecrash.io/
[Real World GitOps with Argo CD and Linkerd]: https://buoyant.io/register/real-world-gitops-with-argo-cd-and-linkerd
[real-world-argo-linkerd]: https://github.com/BuoyantIO/real-world-argo-linkerd
[Linkerd]: https://linkerd.io
[Emissary-ingress]: https://www.getambassador.io/docs/latest/topics/install/install-ambassador-oss/
