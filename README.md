# pi5

GitOps repository for my Raspberry Pi 5 Kubernetes cluster, managed by [Flux CD](https://fluxcd.io/).

## Structure

```
├── apps/                   # Application workloads
├── clusters/
│   └── pi5/
│       └── flux-system/    # Flux CD bootstrap manifests
└── infrastructure/         # Cluster-level infrastructure
```

## GitOps with Flux

Flux is bootstrapped into the `clusters/pi5/flux-system` directory and watches this repository. It reconciles two top-level Kustomizations:

- **infrastructure** - deployed first; sets up cert-manager and Traefik.
- **apps** - deployed after infrastructure is ready; manages all application workloads.

Both Kustomizations run on a 10-minute interval and use SOPS for secret decryption.

## Secrets

Secrets are encrypted in-repo using [SOPS](https://github.com/getsops/sops) with a GPG key. Flux decrypts them at apply time using the `sops-gpg` Kubernetes secret in the `flux-system` namespace.

The SOPS configuration (`.sops.yaml`) encrypts the following fields in any `*.yaml` file:

- `data`, `stringData` - Kubernetes Secret values
- `email`, `apiToken`, `host`, `hosts` - provider credentials / ingress config

To encrypt a secret locally you need the GPG private key imported:

```sh
gpg --import private.asc
```

Then run:

```sh
sops --encrypt --in-place secret.yaml
```

To decrypt a secret locally:

```sh
sops --decrypt --in-place secret.yaml
```

## Bootstrap

Restore the SOPS GPG secret:

```sh
gpg --export-secret-keys --armor 08083A92DBB719D2E10998259B44720A645D6802 | \
  kubectl create secret generic sops-gpg \
    --namespace=flux-system \
    --from-file=sops.asc=/dev/stdin
```

Bootstrap Flux:

```sh
flux bootstrap github \
  --owner=steveiliop56 \
  --repository=pi5 \
  --branch=main \
  --path=clusters/pi5
```

## License

[MIT](LICENSE)
