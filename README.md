# pharmacy-gitops

Kubernetes manifests + FluxCD configuration for the `pharmacy-*` project. Designed for a local cluster (minikube / Docker Desktop / kind) running the `wac-hospital` Polyfea shell.

## Structure

```
apps/
├── xpartla-pharmacy-ufe/       # Stencil micro-frontend (Deployment + Service + Polyfea CRs)
├── xpartla-pharmacy-webapi/    # references xpartla/pharmacy-webapi's own kustomize/install + HTTPRoute + OpenAPI-UI svc
└── mongo-express/              # debug UI for the in-cluster Mongo
components/
├── version-developers/         # image tags auto-updated by Flux
└── version-release/            # fixed semver tags (for shared cluster releases)
infrastructure/
├── fluxcd/        # FluxCD v2.8.1 install
├── polyfea/       # Polyfea controller + http-route + shell config
└── envoy-gateway/ # Envoy Gateway v1.7.1 + GatewayClass + HTTP-only Gateway
clusters/
└── localhost/
    ├── gitops/       # FluxCD Kustomizations + ImageRepo/Policy + ImageUpdateAutomation
    ├── prepare/      # infrastructure/* bundle
    └── install/      # apps/* + components/version-developers + remote mongodb component
```

## Bootstrap

Prerequisites: minikube running, Docker images `xpartla/pharmacy-ufe` and `xpartla/pharmacy-webapi` published with `main.*` tags on Docker Hub.

```bash
# 1. Install Flux into the cluster (image automation controllers required)
flux install --components-extra=image-reflector-controller,image-automation-controller

# 2. Create the namespace
kubectl create namespace wac-hospital

# 3. Point Flux at this repo (this kicks off everything else)
kubectl apply -k clusters/localhost/gitops

# 4. Watch reconciliation
flux get kustomizations -A -w

# 5. (in another shell) expose the gateway
minikube tunnel
```

App will be available at `http://localhost/xpartla-pharmacy-product/`. Add `127.0.0.1 wac-hospital.loc` to `/etc/hosts` if you prefer that hostname.

## Out of scope (intentionally)

This is a simplified gitops repo compared to the ambulance reference. **Not included:** cert-manager / HTTPS, SOPS-encrypted secrets, Dex / OIDC, OPA, observability stack, repository PAT. The cluster runs HTTP-only and the gitops repo is public.
