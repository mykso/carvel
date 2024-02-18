---
title: "Lazy Sync global lazy cache"
authors: [ "Fritz Duchardt <fritz@duchardt.net>" ]
---

# Global Lazy Sync Cache

## Problem Statement
Our rollout mechanism deploys the same workloads to multiple clusters. Most of these workloads have lazy syncing enabled, since they are based on fixed versions. Thanks to lazy syncing, we only fetch each workload once for each cluster. Nevertheless, it would be more efficient, if we had to fetch each workload with the same version number only once for all clusters.

## Proposal
We introduce a global lazy cache for all lazy workloads. 
The cache is stored at a central location and affect all vendir runs on the same machine.
It caches the upstream workloads before they are potentially modified by Vendir settings like `includePaths` in order to maximize the number or cachable workloads.
It is activated and configured with environment variables.

### Goals
Users benefit from quicker syncing process, if they rely on lazy syncing.

### Specification / Use Cases
The following environment variables can be used to configure global lazy caching:
```
# to activate the feature
VENDIR_LAZY_CACHE=true
# cache location
VENDIR_LAZY_CACHE_DIR=~/.vendir/lazycache
```
The directory naming within `lazycache` leverages unique human readable workload identifiers, both to keep track of already cached workloads for Vendir and for the user to tell what has been cached, e.g. during manual clean-up:
```
+ lazycache
  + helm-multus-cni-1.1.10-configdigest
  + directory-multus-configdigest
  + git-multus-cni-ref-configdigest
```
If fetching a lazy workload with global lazy cache enabled, Vendir syncs in the following step:

1. Check whether syncing is required based on the existing lazy logic (configDigest comparison of vendir.yaml and vendir.lock.yaml).
2. If syncing is required, check the global lazy cache first. Copy an existing entry into its staging destination to apply filters like `includePaths` and fill the `vendor` directory. 
3. Only if the global lazy cache is empty, pull the workload from the upstream repo and fill global lazy cache as well as `vendor` directory.

If the user forces a sync despite the `lazy` setting with the `--lazy=false` option, global lazy cache also gets resynced.

## Open Questions

- Do we need a cache cleanup mechanism?

## Answered Questions
