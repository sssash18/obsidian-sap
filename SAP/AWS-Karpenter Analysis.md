### <u>ScaleUp Flow</u>
[[MCM-CA Overhaul POC]]

1. In `pkg/controllers/provisioning/provisioner.go:Schedule` all pending pods (sorted in decreasing order) along with pods that needs to be rescheduled from deleting nodes are gathered in a slice.
2. Then in `pkg/controllers/provisioning/scheduling/scheduler.go:Solve` we iterate on each pod and call `pkg/controllers/provisioning/scheduling/scheduler.go:add`.
3. In `add` we first try to fit the pod on existing nodes and the upcoming `NewNodeClaims`. Otherwise a new `NodeClaim` is created considering the `daemonOverhead.
4. `NodeClaims` initially contains all the `machineTypes`. We then filter the instances out does not satisfy the pod resource requests.
5. After filtering the instance types on all the pods in the loop. we return the results which contain `nodeclaims`
6. At last the instances are truncated if there count > 100 to make it 100.
7. Then after the pendingpods loop all `NewNodeClaims` are created.
8. Now the NodeClaims are reconciled seperately.
9. In `pkg/controllers/nodeclaim/lifecycle/launch.go:Reconcile`.

### <u>Consolidation</u>

#### SingleNodeConsolidation

Try to delete any single node, possibly launching a single replacement whose price is lower than that of the node being removed.

##### Flow
1. We first try to find the possible candidates that can be consolidated.
2. Then we take each candidate and try to fit the pods on it on a new single machine along with the pending pods.
3. Then we sort all possible instances by price and choose the one with lowest pricing and create the `newnodeclaim`.

#### MultiNodeConsolidation

Try to delete two or more nodes in parallel, possibly launching a single replacement whose price is lower than that of all nodes being removed.

##### Flow
1. We first try to find the possible candidates that can be consolidated.
2. Then we binary search on the nodeclaims which we can consolidate at once by calling `computeConsolidation`.
3. Then some filters happens.
4. The command is then actually executed.

