---
created: 2024-01-24
last updated: 2024-01-24
status: To be reviewed
reviewers:
  -
title: Input addressed action cache
authors:
  - Silic0nS0ldier
---

(please note that this is incomplete, terminology in particular is likely to be minced and will be cleaned up before this proposal is put to review)

# Abstract

Complementing the content addressed action cache with an input addressed alias so builds can be short-circuited from the bottom up.


# Background

This section can give context. For example, it can explain the current state and
have pointers to previous design documents.

Because Bazel uses a content addressed approach to caching, it is possible to finish builds early if input changes do not produce output changes. This brings significant savings but has limitations;
- Cache lookup cannot be performed for top level targets without first retrieving information about all its dependencies.
- Remotes are forced to persist infrequently changed action cache entries because they are required to determine if their consumers are cached.
  Perhaps more significantly, they cannot be moved to a cheaper storage medium as it would hurt build performance.

Nix takes a different approach. At present (2024-01-24) it uses an input addressed store which allows the store entry and its references to be pulled from caches directly. Input dependencies (e.g. for build) do not need to exist in caches. Nix's input addressed store is the inspiration for this proposal.


# Proposal

The action cache will gain an alias. An input addressed sibling whose entries point to the content address action cache.

(this following needs work, it is a thought dump)
- File inputs are represented a hash of the action that spawns them.
- Bazel will have the option to work backward from the toplevel target.
  This slashes the number of gRPC calls required to determine if a cached result exists.
- Cache checks can be performed at arbitrary points in the action graph, speeding up partial cache misses (the most common scenario) significantly.
  Since action cache lookups are often batched, Bazel could pick a selection across the graph so that;
  1. Reduce the number of gRPC -> check -> gRPC interactions.
  2. The remote is not swamped by a enormous action cache query (e.g. the entire graph), which induces a lot of load.
- Actions further away from toplevel targets can be evicted more aggressively from the input addressed action cache.
  Especially important as this cache is likely to grow quicker vs. content addressed (action hashes are more coarse vs. file hashes, especially input addressed action hashes so there will be more churn).


# Backward-compatibility

Describe here how this proposal impacts backward compatibility. If the proposal
is implemented, can it possibly break any user?
