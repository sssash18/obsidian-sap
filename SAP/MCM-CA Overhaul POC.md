#Story
#### [POC Project](https://github.com/elankath/scaler-simulator)

[[CA vs POC]] - 01/03/2024

---
### TODO
- [ ] Come up with CA vs POC cases where CA outperforms POC.  [[CA vs POC]]
- [ ] To deduct `daemon sets` and `system-components` resource usage from new virtual nodes `allocatable` in status.

---
### Progress `01-03-2024`
- Added actual scaleup of cluster in shoot reacting on the recommendation of POC.
- Perform cleanup of the scaling actions
  
---
### <u>Notes from Meets</u>
#### KubeScheduler touchpoints in CA code
1. CA in its [HintingSimulator.TrySchedulePods](https://github.com/gardener/autoscaler/blob/1cbbdef2762f73e83515a1296a691039aaf2e456/cluster-autoscaler/simulator/scheduling/hinting_simulator.go#L58)  _first_ attempts to schedule unschedulable pods on any current/upcoming nodes. For each unschedulable pod, it attempts to find an existing node using the scheduler framework code invoked from [SchedulerBasedPredicateChecker.FitsAnyNodeMatching](https://github.com/gardener/autoscaler/blob/1cbbdef2762f73e83515a1296a691039aaf2e456/cluster-autoscaler/simulator/predicatechecker/schedulerbased.go#L96). This guy runs the [scheduler loop](https://github.com/kubernetes/kubernetes/blob/73f19e4c0162a2ee711c254b2af226b4e135d0e3/pkg/scheduler/framework/interface.go#L589) (RunPreFilterPlugins) and gets a result struct that encapsulates the node names that can fit the pod.
2. If there are still unschedulable pods, then we need a scaleup. The CA attempts to scale up the cluster using.  [ScaleupOrchestrator.ScaleUp](https://github.com/kubernetes/autoscaler/blob/0ebbdfb263f573400f470cc76ddcf38d89cc059e/cluster-autoscaler/core/scaleup/orchestrator/orchestrator.go#L84)

	1. The CA  iterates through NodeGroups,  obtains a ‘template’ [NodeInfo](https://github.com/kubernetes/kubernetes/blob/73f19e4c0162a2ee711c254b2af226b4e135d0e3/pkg/scheduler/framework/types.go#L524)  by interrogating [CloudProvider.NodeGroup.TemplateNodeInfo()](https://github.com/gardener/autoscaler/blob/1cbbdef2762f73e83515a1296a691039aaf2e456/cluster-autoscaler/cloudprovider/cloud_provider.go#L213)   then for each pending unschedulable pod, checks to see if the Pod can be assigned on this new template node using [SchedulerBasedPredicateChecker.CheckPredicates](https://github.com/gardener/autoscaler/blob/1cbbdef2762f73e83515a1296a691039aaf2e456/cluster-autoscaler/simulator/predicatechecker/schedulerbased.go#L145) . (This guy  also runs the [scheduler loop](https://github.com/kubernetes/kubernetes/blob/73f19e4c0162a2ee711c254b2af226b4e135d0e3/pkg/scheduler/framework/interface.go#L589) (RunPreFilterPlugins) and gets a result struct that encapsulates the template node node if it can fit the pod or  a nil status if it cant)
	2. Now that the unschedulable pods have been scheduled on template nodes, the CA then computes [expander options](https://github.com/gardener/autoscaler/blob/1cbbdef2762f73e83515a1296a691039aaf2e456/cluster-autoscaler/expander/expander.go#L44) (for each node group) based on the now-schedulable pods using [ScaleUpOrchestrator.ComputeExpansionOption](https://github.com/gardener/autoscaler/blob/1cbbdef2762f73e83515a1296a691039aaf2e456/cluster-autoscaler/core/scaleup/orchestrator/orchestrator.go#L480)
	3. It then picks up the best expander option (and thus selected node group for expansion) using the configured strategy.

		1. If CA is configured with `--balance-similar-node-groups` then it computes  node groups similar to the previously selected node  group using [ScaleUpOrchestrator.ComputeSimilarNodeGroups](https://github.com/kubernetes/autoscaler/blob/6c14a3a3cd1b2976a6dc0d3a9917a11786994959/cluster-autoscaler/core/scaleup/orchestrator/orchestrator.go#L634) .  ie find nodegroups that can schedule the same set of pods as the selected node group of the best expander option

	4. It then executes the scale ups on the target selected node groups.

### Scenarios Outputs

- [[Scenario Outputs]]