Homelab Kubernetes with Flux CD

This repository contains a GitOps setup for a homelab Kubernetes cluster managed with Flux CD. It provisions core infrastructure like Traefik (ingress controller with ACME via Cloudflare DNS01) and some internal services (landing page, Grafana, Portainer, etc.).


Prerequisites
- A running Kubernetes cluster and kubectl configured to talk to it
- Flux CLI installed (https://fluxcd.io)
- Optional: GitHub repository access (SSH) for Flux to sync this repo
- Cloudflare account with a Zone that you control
- Cloudflare API Token with DNS edit permission on the target zone (used for ACME DNS-01)

Important: Cloudflare API token
Traefik is configured to use the environment variable CF_DNS_API_TOKEN sourced from a Kubernetes Secret named cloudflare-credentials in the traefik namespace. Create this Secret before Traefik attempts to obtain certificates.

1) Create the traefik namespace (if it does not already exist)
```shell
kubectl create namespace traefik || true
```

2) Create the Secret with your Cloudflare API token (recommended) or legacy key
```shell
kubectl create secret generic cloudflare-credentials \
  --from-literal=apiKey="YOUR_CLOUDFLARE_API_TOKEN" \
  -n traefik
```

Notes
- The token should have at least Zone:DNS:Edit on the domain(s) you plan to issue certificates for.
- In this repo, the Traefik HelmRelease (clusters/home/infra/traefik/traefik-release.yaml) maps the Secret key apiKey to the env var CF_DNS_API_TOKEN used by lego/Traefik.

Bootstrapping Flux
Option A — Using flux bootstrap (recommended)
This will install Flux and configure it to sync from your GitHub repo. Replace the values with your GitHub org/repo.
```shell
flux bootstrap github \
  --owner=<your-github-username-or-org> \
  --repository=<your-repo-name> \
  --branch=main \
  --path=clusters/home \
  --personal
```

Option B — Apply manifests directly (advanced)
If you already have Flux controllers on the cluster and an SSH deploy key/secret named flux-system, you can apply the sync manifests directly:
```shell
kubectl apply -f clusters/home/infra/flux-system/
```
The gotk-sync.yaml in this repo points to `ssh://git@github.com/kalpak44/homelab-k8s` on the main branch. If you fork or rename the repo, update this URL accordingly or re-bootstrap with Option A.

Traefik and TLS
- Traefik exposes HTTP(S) via LoadBalancer services. By default, it redirects HTTP to HTTPS.
- ACME DNS-01 via Cloudflare is enabled with the certificatesResolver named cloudflare.
- ACME email and other options are set in `clusters/home/infra/traefik/traefik-release.yaml`. Adjust as needed (e.g., email, domains via Ingress/IngressRoute).

Weave GitOps UI (optional)
Manifests are provided under clusters/home/weave-gitops. After Flux applies them, expose and secure access per your environment.

Troubleshooting
- Secret not found: Ensure cloudflare-credentials exists in the traefik namespace and contains the apiKey key before Traefik starts certificate requests.
- Flux not syncing: Check flux-system namespace logs (source-controller, kustomize-controller). Verify repository URL/SSH keys match your repo.
- Certificates pending: Verify the Cloudflare token has the correct Zone:DNS permissions and your domain’s DNS is managed by Cloudflare.
