#   Platform Setup

##  Prepare to deploy management clusters

([docs](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/2.1/tkg-deploy-mc-21/mgmt-reqs-prep-aws.html))

##  Deploy a management cluster

    $ tanzu management-cluster create --ui

##  Create a Workload Cluster

    $ tanzu cluster create multi-tenant-platform --file platform/multi-tenant-platform.yaml

##  Add a Package Repository

    $ tanzu package repository add tanzu-standard --url projects.registry.vmware.com/tkg/packages/standard/repo:v2.1.0 --namespace tkg-system
    $ k create ns tanzu-cli-managed-packages

##  Install Cert-Manager

    $ tanzu package install cert-manager --package cert-manager.tanzu.vmware.com --version 1.7.2+vmware.3-tkg.1 -n tanzu-cli-managed-packages
    $ k apply -f platform/cert-manager/aws-credentials-secret.yaml
    $ k apply -f platform/cert-manager/cluster-issuer-staging.yaml
    $ k apply -f platform/cert-manager/cluster-issuer-prod.yaml

##  Install Contour

    $ tanzu package install contour --package contour.tanzu.vmware.com --version 1.22.3+vmware.1-tkg.1 --values-file platform/contour/values.yaml -n tanzu-cli-managed-packages

##  Install External DNS

    $ k create ns tanzu-system-service-discovery
    $ k apply -f platform/external-dns/aws-credentials-secret.yaml
    $ tanzu package install external-dns --package external-dns.tanzu.vmware.com --version 0.12.2+vmware.4-tkg.1 --values-file platform/external-dns/values.yaml -n tanzu-cli-managed-packages

##  Install Flux Source Controller

    $ GITHUB_TOKEN=... flux bootstrap github --owner=p-ssanders --repository=secure-supply-chain-demo --branch=main --path=gitops --private-key-file=github --personal

##  Install Flux Kustomize Controller

    $

##  Configure Flux Kustomize Controller for GitOps with SOPS/PGP

https://fluxcd.io/flux/guides/mozilla-sops/

    $ gpg --export-secret-keys --armor "${KEY_FP}" | kubectl create secret generic sops-keys --namespace=kustomize-system -o yaml --dry-run=client --from-file=identity.asc=/dev/stdin > platform/flux-kustomize/sops-keys-secret.yaml
    $ k apply -f platform/flux-kustomize/sops-keys-secret.yaml
    $ k apply -f platform/flux-kustomize/gitops-ssh-credentials-secret.yaml
    $ k apply -f platform/flux-kustomize/gitops-pgp-public-keys-secret.yaml
    $ k apply -f platform/flux-kustomize/gitops-gitrepository.yaml
    $ k apply -f platform/flux-kustomize/gitops-kustomization.yaml

##  Install Harbor

    $ imgpkg pull -b projects.registry.vmware.com/tkg/packages/standard/harbor:v2.5.3_vmware.1-tkg.1 -o /tmp/harbor-package-2.5.3+vmware.1-tkg.1
    $ bash /tmp/harbor-package-2.5.3+vmware.1-tkg.1/config/scripts/generate-passwords.sh platform/harbor/values.yaml
    $ tanzu package install harbor --package harbor.tanzu.vmware.com --version 2.5.3+vmware.1-tkg.1 -f platform/harbor/values-secret.yaml
    $ k apply -f platform/harbor/package-install-overlay.yaml
