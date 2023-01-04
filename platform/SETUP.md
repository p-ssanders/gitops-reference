#   Platform Setup

##  Prepare to deploy management clusters

1.  Create an access key
1.  `aws configure --profile tkg-demo`
1.  `export AWS_PROFILE=tkg-demo`
1.  `aws ec2 create-key-pair --key-name tkg-demo --output json | jq .KeyMaterial -r > tkg-demo.pem`

([docs](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.6/vmware-tanzu-kubernetes-grid-16/GUID-mgmt-clusters-aws.html))

##  Deploy a management cluster

    $ tanzu management-cluster create --ui

##  Create a Workload Cluster

    $ tanzu cluster create demo-acceptance-workload --file platform/demo-acceptance-workload.yaml
    $ tanzu cluster create multi-tenant-platform --file platform/multi-tenant-platform.yaml

##  Install Cert-Manager

    $ tanzu package install cert-manager --package-name cert-manager.tanzu.vmware.com --version 1.7.2+vmware.1-tkg.1
    $ k apply -f platform/cert-manager/aws-credentials-secret.yaml
    $ k apply -f platform/cert-manager/cluster-issuer-staging.yaml
    $ k apply -f platform/cert-manager/cluster-issuer-prod.yaml

##  Install Contour

    $ tanzu package install contour --package-name contour.tanzu.vmware.com --version 1.20.2+vmware.1-tkg.1 --values-file platform/contour/values.yaml

##  Install External DNS

    $ k create ns tanzu-system-service-discovery
    $ k apply -f platform/external-dns/aws-credentials-secret.yaml
    $ tanzu package install external-dns --package-name external-dns.tanzu.vmware.com --version 0.11.0+vmware.1-tkg.2 -f platform/external-dns/values.yaml

##  Install Flux Source Controller

    $ tanzu package install fluxcd-source --package-name fluxcd-source-controller.tanzu.vmware.com --version 0.24.4+vmware.1-tkg.4

##  Install Flux Kustomize Controller

    $ tanzu package install fluxcd-kustomize --package-name fluxcd-kustomize-controller.tanzu.vmware.com --version 0.24.4+vmware.1-tkg.1

##  Configure Flux Kustomize Controller for GitOps with SOPS/PGP

https://fluxcd.io/flux/guides/mozilla-sops/

    $ gpg --export-secret-keys --armor "${KEY_FP}" | kubectl create secret generic sops-gpg --namespace=kustomize-system -o yaml --dry-run=client --from-file=sops.asc=/dev/stdin > sops-gpg-secret.yaml
    $ k apply -f platform/flux-kustomize/sops-pgp-secret.yaml
    $ k apply -f platform/flux-kustomize/gitops-ssh-credentials-secret.yaml
    $ k apply -f platform/flux-kustomize/gitops-pgp-public-keys-secret.yaml
    $ k apply -f platform/flux-kustomize/gitops-gitrepository.yaml
    $ k apply -f platform/flux-kustomize/gitops-kustomization.yaml

##  Install Harbor

    $ imgpkg pull -b projects.registry.vmware.com/tkg/packages/standard/harbor:v2.5.3_vmware.1-tkg.1 -o /tmp/harbor-package-2.5.3+vmware.1-tkg.1
    $ bash /tmp/harbor-package-2.5.3+vmware.1-tkg.1/config/scripts/generate-passwords.sh platform/harbor/values.yaml
    $ tanzu package install harbor --package-name harbor.tanzu.vmware.com --version 2.5.3+vmware.1-tkg.1 -f platform/harbor/values-secret.yaml
    $ k apply -f platform/harbor/package-install-overlay.yaml
