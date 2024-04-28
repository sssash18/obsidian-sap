[[MCM-CA Overhaul POC]]
### Observations for `CA`

- CA builds `Podequivalencegroups` based on controller ref and similarity in `pod.Spec`.
- Then it iterates on each `nodeGroup` and tries to create a `option` which says the `podgroups` that can fit on the nodes of this `nodegroup` in `ComputeExpansionOption` and the `scaleup count` required in the `Estimate` function.
  
```
ComputeExpansionOption -> iterates on each nodegroup to calculate podgroups that can fit on the nodes -> Calls Estimate -> this calculates the node count required -> Choose best option for scaleup.
```

---
### Suyash Algo

```
1. We decide to just scaleup one machine of each type in one iteration.
2. Then for each kind of machine we try to fit max pods we can in that machine.
3. We calculate a score for each machine, where score is =:> waste + (unscheduled pods after scaling this machine)/(total unscheduled pods in pool) + (cost of this instance)/(maxcost of all instances).
4. We create virtual node for only that machine with least score and perform the simulation then.

This will also have some suboptimal cases, but would like to know your views on it (edited)
```

#### TODO

- [ ] Order the winners and then select serially whichever is possible. 
- [ ] TSC is not respected in virtual cluster.
### Cases comparing CA over POC over algos

#### Case 1

```
PodA : 3Gb -> 3 replicas
NG1 : m5.large -> 8Gb; NG1Max : 4
NG2 : m5.xlarge -> 16Gb; NG2Max : 4
Result of POC -> 2 * NG1
Result of CA -> 1 * NG2
Result of suyash's alg -> 
```

#### Case 2

```
PodA : 5Gb -> 10 Replicas
NG1 : m5.large -> 8Gb; NG1Max: 12
NG2 : m5.4xlarge -> 64Gb; NG2Max: 4
Result of POC -> 10 * NG1
Result of CA -> 1 * NG2
```

#### Case 3 (Even if we deploy larger pod first)

```
PodA : 5Gb -> 10 Replica
PodB : 12Gb -> 1 Replica
NG1 : m5.large -> 8Gb; NG1Max: 12
NG2 : m5.xlarge -> 16Gb; NG2Max: 5
NG3 : m5.4xlarge -> 64Gb; NG3Max: 5

Result of CA -> 1 * NG3

###score algo
#run 1
order machines by smallest to largest
pods by smallest to largest

result : 10 * NG1, 1 * NG2

#run 2
order machines by smallest to largest
pods by largest to smallest

result : 1 * NG2, 10 * NG1

#run 3
order machines by largest to smallest
pods by smallest to largest

result : 1 * NG3

#run 4
order machines by largest to smallest
pods by largest to smallest

result : 1 * NG3
```

#### Case 4

```
PodA : 4GB -> 2 Repl
PodB : 10GB -> 2 Repl
NG1 : m5.large -> 8GB; NG1Max:  5
NG2 : m5.xlarge -> 16Gb; NG2Max: 5
NG3 : m5.4xlarge -> 64Gb; NG3Max: 5
Result of POC -> 2*NG1 , 2*NG2 (s -> l)
1*NG3 (l -> s)
Result(optimal) -> 2 * NG2

###score algo
#run 1
order machines by smallest to largest
pods by smallest to largest

result : 2 * NG1, 2 * NG2

#run 2
order machines by smallest to largest
pods by largest to smallest

result : 2 * NG2

#run 3
order machines by largest to smallest
pods by smallest to largest

result : 1 * NG3

#run 4
order machines by largest to smallest
pods by largest to smallest

result : 1 * NG3

Result(score-algo) ->  2 * NG2


```

#### Case 5

```
PodA : 7GB -> 2 Repl; 
PodB : 10GB -> 2 Repl; 
NG1 : m5.large -> 8GB; NG1Max:  5
NG2 : m5.xlarge -> 16Gb; NG2Max: 5
NG3 : m5.4xlarge -> 64Gb; NG3Max: 5
Result of POC -> 2*NG1 , 2*NG2 (s -> l)
1*NG3 (l -> s)
Result(optimal) -> 2 * NG1, 2 * NG2

###score algo
#run 1
order machines by smallest to largest
pods by smallest to largest

result : 2 * NG1, 2 * NG2

#run 2
order machines by smallest to largest
pods by largest to smallest

result : 2 * NG1, 2 * NG2

#run 3
order machines by largest to smallest
pods by smallest to largest

result : 1 * NG3

#run 4
order machines by largest to smallest
pods by largest to smallest

result : 1 * NG3

Result(score-algo) -> 2 * NG1, 2 * NG2
```

#### Case 6

```
PodA : 7GB -> 2 Repl; TSC : node, maxSkew : 1
PodB : 10GB -> 2 Repl; TSC : node, maxSkew : 1
NG1 : m5.large -> 8GB; NG1Max:  5
NG2 : m5.xlarge -> 16Gb; NG2Max: 5
NG3 : m5.4xlarge -> 64Gb; NG3Max: 5
Result of POC -> 2*NG1 , 2*NG2 (s -> l)
1*NG3 (l -> s)
Result(optimal) -> 2 * NG1, 2 * NG2

###score algo
#run 1
order machines by smallest to largest
pods by smallest to largest

result : 2 * NG1, 2 * NG2

#run 2
order machines by smallest to largest
pods by largest to smallest

result : 2 * NG1, 2 * NG2

#run 3
order machines by largest to smallest
pods by smallest to largest

result : 2 * NG3

#run 4
order machines by largest to smallest
pods by largest to smallest

result : 2 * NG3

Result(score-algo) -> 2 * NG1, 2 * NG2

###Suyash algo
small -> large pod order
#run 1
ng1 : 1/8 + 3/4 + 1/4 -> 1.1
ng2 : 9/16 + 3/4 + 2/4 -> 1.81
ng3 : 47/64 + 2/4 + 1 -> 2.23
winner : ng1
rem pods : 1a,2b
#run 2
ng1 : 1/8 + 2/3 + 1/4 -> 1.04
ng2 : 9/16 + 2/3 + 2/4 -> 1.72
ng3 : 47/64 + 1/3 + 1 -> 2.06
winner: ng1
rem pods : 2b
#run 3
ng2 : 6/16 + 1/2 + 2/4 => 1.37
ng3 : 44/64 + 0 + 4/4 => 1.68
winner: ng2
rempods: 1b
#run 4
winner:  ng2
result -> 2 * NG1, 2 * NG2 (optimal)
```

#### Case 7

```
PodA : 3Gb -> 10Repl
NG1 : m5.large -> 8GB; NG1Max:  5
NG2 : m5.2xlarge -> 32Gb; NG2Max: 5
NG3 : m5.4xlarge -> 64Gb; NG3Max: 5

Result(optimal) -> 1 * NG2

###score algo
#run 1
order machines by smallest to largest
pods by smallest to largest

result : 5 * NG1

#run 2
order machines by smallest to largest
pods by largest to smallest

result : 5 * NG1

#run 3
order machines by largest to smallest
pods by smallest to largest

result : 1 * NG3

#run 4
order machines by largest to smallest
pods by largest to smallest

result : 1 * NG3

Result(score-algo) -> No Optimal Solution
```

#### Case 8

```
PodA : 3Gb -> 15Repl
NG1 : m5.large -> 8GB; NG1Max:  1
NG2 : m5.xlarge -> 16GB; NG2Max : 2
NG3 : m5.2xlarge -> 32Gb; NG3Max: 5
NG4 : m5.4xlarge -> 64Gb; NG4Max: 5

Result(optimal) -> 1 * NG2, 1 * NG3

###score algo
#run 1
order machines by smallest to largest
pods by smallest to largest

result : 1 * NG1, 2 * NG2, 1 * NG3

#run 2
order machines by smallest to largest
pods by largest to smallest

result : 1 * NG1, 2 * NG2, 1 * NG3

#run 3
order machines by largest to smallest
pods by smallest to largest

result : 1 * NG4

#run 4
order machines by largest to smallest
pods by largest to smallest

result : 1 * NG4

Result(score-algo) -> No Optimal

Result of CA 

#run 1
op 1 : 2/8
op 2 : 2/32
op 3 : 19/64
op 4 : 19/64
chosen : op2
Rem pods : 5
Scaleup : 2 * NG2

#run 2
op1 : 2/8
op2 : NA
op3 : 17/32
op4 : 49/64
chosen : op1
Rem pods : 3
Scaleup : 1 * NG1

#run 3
Scaleup : 1 * NG3

result -> Not Optimal (same as POC)

PodA : 3Gb -> 15Repl
NG1 : m5.large -> 8GB; NG1Max:  1
NG2 : m5.xlarge -> 16GB; NG2Max : 2
NG3 : m5.2xlarge -> 32Gb; NG3Max: 5
NG4 : m5.4xlarge -> 64Gb; NG4Max: 5

## Suyashalgo run
#run 1
ng1: 2/8 + 13/15 + 1/4=> 1.36
ng2: 1/16 + 12/15 + 2/4=> 2.36
ng3: 2/32 + 5/15 + 3/4=> 1.14
ng4: 19/64 + 0/15 + 1 => 1.29
winner:  ng3
rem pods : 5

#run 2
ng1: 2/8 + 3/5 + 1/4=> 1.1
ng2: 1/16 + 0/5 + 2/4=> 0.56
ng3: 17/32 + 0/5 + 3/4=> 1.28
ng4: 49/64 + 0/5 + 1 => 1.76
winner:  ng2
rem pods : 0

result : 1 * NG2 , 1 * NG3 (optimal)
```

#### Case 9

```
PodA: 5GB -> 10 replicas  
   TSC: topology-key = `zone`, maxSkew=1, whenUnsatisfiable=`DoNotSchedule`  
PodB: 12GB -> 2 replicas  
   TSC: topology-key = `node`, maxSkew = 1, whenUnsatisfiable=`DoNotSchedule`  
NG1 -> m5.Large -> Zone-A (8gb)  (Max : 12)
NG2 -> m5.xLarge -> Zone-B (16 gb)  (Max : 5)
NG3 -> m5.4xLarge -> Zone-C (64 gb) (Max : 5)

Result (optimal) : 3 * NG1, 2 * NG2, 1 * NG3

#run 1
order machines by smallest to largest
pods by smallest to largest

result : NG11,NG21,NG31, NG12,NG21,NG31, NG13, NG21, NG31, NG14, NG22, NG23
4 * NG1, 3 * NG2, 1 * NG3

#run 2
order machines by smallest to largest
pods by largest to smallest

result : NG21, NG22, NG11, NG23, NG31, NG12, NG23, NG31, NG13, NG23, NG31, NG14
4 * NG1, 3 * NG2, 1 * NG3

#run 3
order machines by largest to smallest
pods by smallest to largest

result : NG31, NG21, NG11, NG31, NG21, NG12, NG31, NG21, NG13, NG31, NG31, NG32
2 * NG3, 2 * NG2, 3 * NG1

#run 4
order machines by largest to smallest
pods by largest to smallest

result : NG31, NG32, NG....
2 * NG3, 2 * NG2, 3 * NG1

###Suyashalgo
PodA: 5GB -> 10 replicas  
   TSC: topology-key = `zone`, maxSkew=1, whenUnsatisfiable=`DoNotSchedule`  
PodB: 12GB -> 2 replicas  
   TSC: topology-key = `node`, maxSkew = 1, whenUnsatisfiable=`DoNotSchedule`  
NG1 -> m5.Large -> Zone-A (8gb)  (Max : 12)
NG2 -> m5.xLarge -> Zone-B (16 gb)  (Max : 5)
NG3 -> m5.4xLarge -> Zone-C (64 gb) (Max : 5)

small -> large
#run1
ng1 : 3/8 + 11/12 + 1/4 => 1.54
ng2 : 11/16 + 11/12 + 2/4 => 2.10
ng3 : 47/64 + 10/12 + 4/4 => 2.57
winner:  ng1
rempods : 9a,2b

#run2 (ng1 ignored for tsc)
ng2 : 11/16 + 10/11 + 2/4 => 2.10
ng3 : 47/64 + 9/11 + 4/4 => 2.57
winner:  ng2
rempods : 8a,2b
 
#run3
ng3 : ---
winner:  ng3
rempods : 5a,1b

till now:
ng3 : 2a 1b
ng2 : 2a 
ng1 : 1a

#run4
ng1 : 3/8 + 4/7 + 1/4 => 1.20
ng1 -> 1a , ng2 -> 1a , ng3 -> 1a
rem: 3a 1b,
ng2 : 4/16 + 6/7 + 2/4 => 1.6
ng2 -> 1b
rem : 6a
ng3 : 52/64 + 6/7 + 4/4 => 2.67
ng3 -> 1b
rem: 6a
winner:  ng1
rempods : 2a 1b

till now:
ng3 : 3a 1b (1 nodes)
ng2 : 3a (1)
ng1 : 2a (2)

Rough:
ng3 : 3a 1b (1 nodes)
ng2 : 2a (1)
ng1 : 2a (2)

Rough:
ng3 : 3a 1b (1 nodes)
ng2 : 2a (1)
ng1 : 3a (3)

Rough:
ng3 : 3a 1b (1 nodes)
ng2 : 2a + 2a (2)
ng1 : 3a (3)

Rough:
ng3 : 3a 1b (1 nodes)
ng2 : 2a + 2a (3)
ng1 : 3a (3)

#run5
ng1 : 3/8 + 2/4 + 1/4 => 1.125
ng1 -> 1a, ng3 -> 1a
ng2 : 4/16 + 3/4 + 2/4 => 1.5
ng2 -> 1b
ng3 : 52/64 + 3/4 + 4/4 => 2.56
ng3 -> 1b
winner : ng1
rem pods :  1b

till now:
ng3 : 4a 1b (1 nodes)
ng2 : 3a (1)
ng1 : 3a (3)

#run6
ng2 : 4/16 + 3/4 + 2/4 => 1.5
ng2 -> 1b
ng3 : 52/64 + 3/4 + 4/4 => 2.56
ng3 -> 1b
winner : ng2
rem pods : 0

till now:
ng3 : 4a 1b (1 nodes)
ng2 : 3a (2)
ng1 : 3a (3)

result -> 3 * NG1, 2 * NG2, 1 * NG1 (optimal)

CA result -> 3p1, 4p2, 1p3 (large to small deploy)
3p1, 3p2, 1p3 (small to large deploy)
```

#### Case 10

```
PodA : 5Gb -> 15Repl
NG1 : m5.large -> 8GB; NG1Max:  20
NG2 : m5.xlarge -> 16GB; NG2Max : 20
NG3 : m5.2xlarge -> 32Gb; NG3Max: 20
NG4 : m5.4xlarge -> 64Gb; NG4Max: 20

Result(optimal) -> 1 * NG2, 1 * NG4

#run 1
order machines by smallest to largest
pods by smallest to largest

result : 15 * NG1

#run 2
order machines by smallest to largest
pods by largest to smallest

result : 15 * NG1

#run 3
order machines by largest to smallest
pods by smallest to largest

result : 2 * NG4

#run 4
order machines by largest to smallest
pods by largest to smallest

result : 2 * NG4

Result(score-algo) -> No Optimal

```

### TODO: Further Scenarios to consider for simulator:  
  - [ ] Scale-Up optimization like described up (deploy heavy pods first)  
  - [ ] Scale from Zero.  
  - [o] Support CA-"analogous": Priority Expander. (Declarative Priority)
  - [ ] Scale-Down scenario below threshold.  
    - [ ] throwing away machine with pod  
    - [ ] we can simulate this and see if pods get scheduled elsewhere.  
  - [ ] Simulate NodeGroup Backoff Handling:  
    - [ ] We add/remove the virtual Node depending on machine health. simulator can run its loop on virtual cluster again and advise new scale-up recommendation.
  - [ ] Handling of DaemonSets and system-components (kubelet,etc) resource deductions for new virtual nodes.
  - [ ] Load testing for big clusters.
### Why not go by kube-scheduler:
- One more point in favour of achieving optimisation at consolidation is that suppose even if we come up with optimal set of scaleup of machines, but actual result will still depend on what profile kubescheduler is using and it takes pods also in order of deployment. So even if we give the optimal set some pods may be unscheduled , and require more scale ups in next iteration. However if we go by consolidation we can control the order on real cluster also by how we evict pods.
- [[CA vs POC#Cases favoring CA over POC]]

### CA ScaleDown
`cluster-autoscaler/simulator/cluster.go:145`
it tries to scheduled pods using `TrySchedulePods` and considers all factors.


### Implementation

1. `ScoreCalculator(ng )` , `CalculateWasteScore`, `CalculateUNSCScore`,`CalculateCostScore`.

### ScaleDown
1. Scale down nodes one by one, as evicting all pods of scaled down nodes can lead to some unscheduled pods.
2. Scaling down one by one can lead to scaledown before cluster state changes.
3. Evict pods in same order as it happens on virtual cluster to prevent undesired unshedulable pod events.
4. In virtual cluster evict pods , randomly each time.