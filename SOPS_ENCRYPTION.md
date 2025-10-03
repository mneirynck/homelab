# Generate the YAML manifest

First, let's create our Cloudflare secrets file with the following command:

```
kubectl -n cloudflare-operator-system create secret generic cloudflare-secrets --from-literal CLOUDFLARE_API_TOKEN=<YourApiToken> --from-literal CLOUDFLARE_API_KEY=<YourAPiKey> --dry-run=client -o yaml > cf-secret.yaml
```

This command will create that secret as a YAML file with the values already encoded. However, these secrets should not be committed to your Git repository in their current state.

# Encrypt your secrets file

Use the configuration to encrypt the secret, first change directory to clusters/homelab :

```
cd clusters/homelab
```

```
sops --encrypt --in-place ../../infrastructure/homelab/cloudflare-operator/cf-secre
```

# Decrypt secrets in K3S

We need to let FluxCD know that we have a private key and that it can be used to decrypt secrets. First, add the key to the cluster:

```
cat age.agekey |
kubectl create secret generic sops-age \
--namespace=flux-system \
--from-file=age.agekey=/dev/stdin
```

Now, tell FluxCD to use this secret for decryption by adding the following to clusters/homelab/infrastructure.yaml:

```
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure
  namespace: flux-system
spec:
  interval: 10m0s
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/homelab
  prune: true
  wait: true
  timeout: 5m0s
  decryption:
    provider: sops
    secretRef:
      name: sops-age
```

And add the cf-secret.yaml file into your kustomization.yaml file under infrastructure/homelab/cloudflare-operator :

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: cloudflare-operator-system
resources:
  - cf-secret.yaml
  - ../../base/cloudflare-operator
  - cluster-tunnel.yaml
```
