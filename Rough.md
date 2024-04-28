/Users/i585976/go/src/github.com/gardener/machine-controller-manager
/Users/i585976/Downloads/suyash-control.yaml
/Users/i585976/Downloads/suyash-target.yaml
/Users/i585976/go/src/github.com/gardener/machine-controller-manager-provider-aws/dev/machineclass_v1.yaml




Worker Pool A -> 8Gb
Worker Pool B -> 32Gb
- Suppose 8 pods needs to be scheduled
- 24Gb  
- Leastwaste fit is 3 nodes from A, but actually it will take 1 B

Hi Vedran,

So CA basically creates *options* for each nodegroup. The option majorly consist of nodegroup, nodecount (required for scaleup) and pods that can be scheduled on this nodegroup. Then it selects on the best option amongst them based on the strategy (lets take least waste here).

(Ignoring daemon sets and other system-components)
So in the example that you took, CA in first iteration will have two options:
1. (11) 5GB pods scheduled on (11) 8GB machines (No 12 GB pod can be scheduled here). -> Waste : 33/88 GB => 37.5 % memory
2. (10) 5GB pods and (1) 12 GB pod scheduled on (1) 64GB machine. -> Waste : 2/64 GB => 3.125 % memory.

So in the first iteration 2nd option will be chosen due to lower waste and in the second iteration CA will schedule the remaining (1) 5GB pod on a new 8GB machine. So in total (1) 8GB and (1) 64 GB machine will scale up in two iterations of CA , which is the optimal solution for this case.

POC results -> 11 * (8GB) , 1 * (64GB)
CA results -> 


```
I0305 05:17:54.052539       1 waste.go:55] Expanding Node Group shoot--scalesim--scenario-c-p3-z1 would waste 95.00% CPU, 37.50% Memory, 66.25% Blended
I0305 05:17:54.052560       1 waste.go:55] Expanding Node Group shoot--scalesim--scenario-c-p4-z1 would waste 95.94% CPU, 47.66% Memory, 71.80% Blended
I0305 05:17:54.052569       1 orchestrator.go:185] Best option to resize: shoot--scalesim--scenario-c-p3-z1
I0305 05:17:54.052576       1 orchestrator.go:189] Estimated 11 nodes needed in shoot--scalesim--scenario-c-p3-z1
I0305 05:17:54.052617       1 orchestrator.go:253] No similar node groups found
I0305 05:17:54.052638       1 orchestrator.go:295] Final scale-up plan: [{shoot--scalesim--scenario-c-p3-z1 0->11 (max: 13)}]
```

Hi Vedran,  
  
We performed the experiment for the scenario you mentioned on CA and POC. Here are the outcomes:  
  
**POC  
**  
As expected (11) m5.large(8GB) machines and (1)m5.4xlarge(64GB) were scaled up. For each smaller pod(5GB) one instance of m5.large was required and for bigger pod(12GB) one instance of m5.4xlarge was scaled.  
  
**CA  
  
**

CA scaled (11) m5.large instances in first iteration and (1) m5.4xlarge instance in second iteration. Here is why it happened:  
Firstly, CA iterates on each nodegroup and [filters](https://github.com/gardener/autoscaler/blob/0ebbdfb263f573400f470cc76ddcf38d89cc059e/cluster-autoscaler/core/scaleup/orchestrator/orchestrator.go#L149) out the pods that can’t be scheduled on it due to resource constraints.Then CA creates **[options](https://github.com/gardener/autoscaler/blob/0ebbdfb263f573400f470cc76ddcf38d89cc059e/cluster-autoscaler/core/scaleup/orchestrator/orchestrator.go#L480)** for each nodegroup with that filtered pod list. The option majorly consist of nodegroup, nodecount (required for scaleup) and pods that can be scheduled on this nodegroup. Then it selects on the [best-option](https://github.com/gardener/autoscaler/blob/0ebbdfb263f573400f470cc76ddcf38d89cc059e/cluster-autoscaler/core/scaleup/orchestrator/orchestrator.go#L177) amongst them based on the strategy (least waste  by default for gardener).  
  
For the first iteration CA has two options:-  
a) (11) m5.large to schedule (11) 5GB pods -> Waste => 33/88 GB = 37.5% (the bigger pod is not part of filtered pod list as it can’t fit on this nodegroup).

b) (2) m5.4xlarge to schedule all pods -> Waste => (11*5 + 12)/128 GB = 47.6%  
  
CA goes with option (a) due to least waste strategy. Then in the second iteration it will scaleup (1) m5.4xlarge  to fit the larger pod(12 GB).  
  
Regards,  
Suyash


```
NodPodAssignment:  pod=scenario-c-small-6xh42 node=p1-xnmrm memory=5368709120000

 Computed node score. nodeScore="(Node: p1-xnmrm, WasteRatio: 0.22, UnscheduledRatio: 0.92, CostRatio: 0.07, CumulativeScore: 1.20)"

 Computed node score. nodeScore="(Node: p2-msqz6, WasteRatio: 0.16, UnscheduledRatio: 0.92, CostRatio: 0.13, CumulativeScore: 1.21)"

Computed node score. nodeScore="(Node: p3-prndj, WasteRatio: 0.08, UnscheduledRatio: 0.67, CostRatio: 0.27, CumulativeScore: 1.02)"

Computed node score. nodeScore="(Node: p4-zrdg2, WasteRatio: 0.50, UnscheduledRatio: 0.50, CostRatio: 0.53, CumulativeScore: 1.54)"
```


```
Recommendation: WorkerPoolName: p1, Replicas: 8, Cost: 145.6320 , Waste: 11581088Ki
WorkerPoolName: p2, Replicas: 1, Cost: 145.6660 , Waste: 973136Ki
TotalCost: 291.2980, TotalWaste: 12554224Ki, TotalAllocatable: 117411824Ki, TotalWasteRatio: 0.1069
```


```
#run1
small-67d10549 -> p1-bcm4d

#run2
large-2026d06b -> p2-4s7qh

#run3
large-f33aefba -> p2-nkjv6

#run4
small-3936e934 -> p2-nb9fb

#run5
small-31c7e42f -> p3-fdv68
small-405a4fe2 -> p3-fdv68
small-f60e7c81 -> p2-nb9fb

#run6
small-996122fe -> p3-fdv68
small-e6877993 -> p2-nb9fb
small-f60e7c81 -> p1-vgkjw

#run7
small-07830a0c -> p3-fdv68
small-996122fe -> p1-28tf4
small-e6877993
```

- Create a new chartapplier and follow the same sequence of mcc as done in gep-aws along with the same kubeconfig


Initial Nodes in the cluster: [p1-f8x96 p1-qqzjp p2-2pgcq p2-45pdv p2-9577g p2-lk47w]
Recommendation for Scale-Down: [p1-qqzjp p1-x4nb4]