"# Argo CD GitOps Bootstrap

This repository contains GitOps manifests for Argo CD and sample applications.

## What is included

- `infrastructure/argocd/kustomization.yaml` - Bootstraps Argo CD in the cluster from the official upstream manifest.
- `infrastructure/namespace.yml` - Creates the target `redis` namespace for the sample applications.
- `infrastructure/argo-apps.yaml` - Argo CD `Application` resources to deploy Bitnami Redis, Redis Cluster, PostgreSQL HA, and PgAdmin.
- `.github/workflows/bootstrap-argo-cd.yml` - GitHub Actions workflow to install Argo CD and apply the GitOps manifests on every push to `main`.

## End-to-end setup

1. Create a Kubernetes cluster and confirm `kubectl` can access it.
2. Add the cluster kubeconfig to GitHub Secrets as `KUBE_CONFIG_DATA`.
   - Encode your kubeconfig with `base64 --wrap=0 ~/.kube/config`.
   - Add the encoded value to the repository secret.
3. Push this repository to GitHub on the `main` branch.
4. The workflow in `.github/workflows/bootstrap-argo-cd.yml` will run on each push and install Argo CD plus the sample applications.

## GitHub Action behavior

The workflow does these steps:

- Checks out the repository.
- Decodes `KUBE_CONFIG_DATA` into a kubeconfig file.
- Installs Argo CD using `kubectl apply -k infrastructure/argocd`.
- Applies the `infrastructure/namespace.yml` and `infrastructure/argo-apps.yaml` manifests.

## Accessing the Argo CD UI

After Argo CD is installed, expose the UI with port-forwarding:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open:

```text
https://localhost:8080
```

## Initial admin credentials

Retrieve the Argo CD initial admin password with:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
```

The username is `admin`.

## Using the Git repository

The repository URL is:

```text
https://github.com/venkatasv/argocdsynctest.git
```

After making changes to the manifests, commit and push them:

```bash
git add .
git commit -m "Bootstrap Argo CD GitOps"
git push origin main
```

Argo CD will sync the updated applications automatically.

## Notes

- If you want to run locally before pushing, use `kubectl apply -k infrastructure/argocd` and then apply the application manifests.
- If you are using a local cluster such as `kind` or `minikube`, make sure the kubeconfig is the same one uploaded to GitHub Secrets.
"
