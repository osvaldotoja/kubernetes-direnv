# Introduction

This repository maps kubernetes resources to the filesystem.

This allows you to navigate through clusters and namespaces via the cli. Using familiar commands such as `cd` and `ls`.

Context is automatically setup so you'll be ready to start typing commands as soon as you `cd`'d into the namespace you're targeting.

# Requirements

* [`direnv`](https://direnv.net/)
* [AWS CLI](https://aws.amazon.com/cli/)
* Access to kubernetes cluster API endpoint.

Current code makes the following assumptions:

* AWS EKS using IAM for authentication.
* All clusters are in `us-east-1` AWS region.
* `awscli` profile is `default`. Please change it accordingly.

Check the FAQ for an example on how to use more regions.

# Usage

```sh
cd us-east-1

# create or update the list of available clusters
update-clusters

# check existing clusters
ls

# select working cluster
cd "cluster-name"

# create or update the list of available namespaces
update-ns

# check existing namespaces
ls

# select working namespace
cd "namespace-name"
```

Work using available commands


```sh
# Print list of pods
kubectl get pods

# Print all logs
stern ".*"
```

Or custom ones, from the `scripts` folder:

```sh
# Create manifest files for resources in the namespace
get-manifests
```

Infrastructure can change. The scripts are not meant to execute a two way synchronization. In order to refresh the list of resources (clusters and/or namespaces) the recommended approach is to regenerate the list of resources. Delete all and generate it again.

For example, for namespaces in a cluster:

```sh
cd $cluster/
rm -fr *
update-ns
```

By using `*` with the `rm` command, we are keeping the `.envrc` file. The one responsible for handling cluster's context. We can safely delete all current folders and let the script download the list of current namespces from the cluster.

# Scripts

| Name | Description |
| --- | --- |
| `update-clusters` | Creates a directory for each cluster in the region. |
| `update-ns` | Creates a directory for each namespace in the cluster. |
| `ns-status` | Print a custom list of namespaces. Helps checking out age. |
| `vpc-status` | Prints associated subnets and available ip counts for each. Useful when monitoring EKS IP exhaustion. |
| `app-status` | Monitor resource status. |
| `pod-status` | Prints pods in trouble. |
| `cleanup-ns` | Example script where kubernetes operations are coupled with external actions. |

# Configuration

Kubeconfig files are stored in the `HOME` directory using the following syntax: "$HOME/.kube/config.${CLUSTER_NAME}.yml"


## FAQ

### What about clusters in multiple regions?

Create a folder for each region (it's not automated) with the same `.envrc` file.

Check example bash code as reference:

```sh
for AWS_REGION in us-east-1 us-east-2; do
  mkdir "${AWS_REGION}"
  cat <<'_EOF_' >"${AWS_REGION}/.envrc"
source_up

AWS_DEFAULT_REGION="$(basename "$(pwd)")"
export AWS_DEFAULT_REGION
_EOF_
  # uncomment this to store regions in git
  #printf '!%s/n' "${AWS_REGION}" >> .gitignore
  printf '/*\n/*/\n!/.envrc\n!.gitignore\n' > "${AWS_REGION}"/.gitignore
done
```

### Which provider is supported?

Current scripts target AWS EKS but it's simple to update for any other Cloud/Provider. First version was written for Rancher 1.0 running Cattle.


### Can I use this tool if I don't have access via `kubectl` to the cluster?

Similar to: will this work in a cluster with no public API endpoints?

As you as you have access to cluster's API endpoint it will work, otherwise it won't.

Those using Rancher to manage their fleet of clusters, this works fine using Rancher API and provided cluster tokens.

### What if I want to use a web UI?

Use [`octant`](https://octant.dev/).

```sh
cd $cluster/$namespace
octant
```

The browser will load with current namespace already selected.

> MacOS users can install it via `brew install octant`
