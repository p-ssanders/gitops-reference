#   GitOps Reference

This repository contains configuration to demonstrate a kubernetes platform that is managed by GitOps using Flux.

The platform setup guide is [here](platform/README.md).

##  Features

The platform is made aware of an application's repository through an out-of-scope onboarding process that results in the creation of:

1.  A git repository for the application's source code
1.  A read-only SSH deploy key to access the git repository
1.  PGP keys for the authors of the application's source code
1.  A subdirectory in this repository under the `gitops` directory

The `gitops` root directory is watched by a a Flux `Kustomization` that automatically synchronizes any changes to subdirectories to the platform, including pruning objects that are not declared in the source.

For each subdirectory (i.e.: application), Flux automatically creates:

1.  A namespace for the application
1.  A Flux `GitRepository` in the application's namespace that watches for changes in the application's repository